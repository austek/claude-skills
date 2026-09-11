---
title: Write In-Memory Interpreters for Repository and Client Traits
impact: HIGH
impactDescription: gives every test a full, real implementation of the algebra instead of a per-test pile of stubbed return values
tags: [mocking, fakes, tagless-final, interpreters]
---

# Write In-Memory Interpreters for Repository and Client Traits [HIGH]

## Description
For a trait that abstracts over persistence or an external client (an "algebra" in tagless-final style — `OrderRepository`, `PaymentGateway`), write one shared in-memory interpreter that fully implements every method against a simple in-process data structure, and have every test that needs the trait construct an instance of that interpreter instead of stubbing individual method calls per test. `class InMemoryOrderRepository(state: Ref[IO, Map[OrderId, Order]]) extends OrderRepository[IO]` implements `save`, `findById`, and `delete` the same way the real Postgres-backed implementation does — just against a `Map` instead of a table — so tests exercise real find-after-save semantics instead of a hand-stubbed answer for each call.

This pays off compounding across a test suite: a new test that needs an `OrderRepository` reuses the existing interpreter instead of writing new stubbing boilerplate, and a change to the trait's contract (a new method, a changed error case) is caught at compile time in the interpreter rather than silently missing from every ad-hoc stub. Keep the interpreter's invariants faithful to production — if `save` on a duplicate ID overwrites in the real implementation, the in-memory one should overwrite too, not silently diverge.

## Bad Example
```scala
test("confirms an order") {
  val repo = mock[OrderRepository[IO]]
  when(repo.findById(orderId)).thenReturn(IO.pure(Some(pendingOrder)))
  when(repo.save(any[Order])).thenReturn(IO.unit)
  // every test re-stubs the same two calls; a third method added to the trait needs
  // a new stub added to every existing test that happens to trigger it
}
```

## Good Example
```scala
final class InMemoryOrderRepository(state: Ref[IO, Map[OrderId, Order]]) extends OrderRepository[IO]:
  def save(order: Order): IO[Unit] = state.update(_ + (order.id -> order))
  def findById(id: OrderId): IO[Option[Order]] = state.get.map(_.get(id))

object InMemoryOrderRepository:
  def withOrder(order: Order): IO[InMemoryOrderRepository] =
    Ref.of[IO, Map[OrderId, Order]](Map(order.id -> order)).map(new InMemoryOrderRepository(_))

class OrderServiceSuite extends munit.CatsEffectSuite:
  test("confirms an order") {
    for
      repo  <- InMemoryOrderRepository.withOrder(pendingOrder)
      _     <- OrderService(repo).confirm(pendingOrder.id)
      saved <- repo.findById(pendingOrder.id)
    yield assertEquals(saved.map(_.status), Some(OrderStatus.Confirmed))
  }
```

## Notes
- `cats.effect.Ref` keeps the interpreter's state update atomic and avoids a bare mutable `var`, matching [`concurrency-avoid-shared-mutable-state`](../../scala-coding-standards/rules/concurrency-avoid-shared-mutable-state.md) from scala-coding-standards.
- One interpreter per trait, shared across the whole test suite — not a fresh hand-rolled fake per test file.
- `fixtures-factory-methods-for-test-data`'s factory-method pattern composes naturally with an interpreter's constructor (`InMemoryOrderRepository.withOrder(sampleOrder())`).

## References
- [cats-effect — Ref](https://typelevel.org/cats-effect/docs/std/ref)
