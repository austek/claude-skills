---
title: Avoid Mockito on Pure, Effect-Typed Code
impact: MEDIUM
impactDescription: avoids stubbing framework machinery fighting final case classes and effect-wrapped return types
tags: [mocking, mockito, effect-systems, pure-functions]
---

# Avoid Mockito on Pure, Effect-Typed Code [MEDIUM]

## Description
Mockito generates a subclass at runtime to intercept method calls, which sits awkwardly against idiomatic Scala 3: `final case class`es and `sealed trait`s can't be subclassed at all, so nothing about them is mockable, and a method returning `IO[Option[Order]]` needs `thenReturn(IO.pure(Some(order)))` — the stub has to know about and construct the wrapping effect correctly, with no compiler check that the stubbed effect matches what the real implementation would actually do (fail, retry, run asynchronously). For a component built the way [`effect-systems-io-for-side-effects`](../../scala-coding-standards/rules/effect-systems-io-for-side-effects.md) and [`error-handling-either-for-domain-errors`](../../scala-coding-standards/rules/error-handling-either-for-domain-errors.md) recommend — pure functions and explicit `Either`/`IO` return types — mocking adds an unsafe, reflection-based layer where a plain fake or a literal function value both work directly against the same types the production code uses.

Where a collaborator is a single-method trait, a plain function value (`OrderId => IO[Option[Order]]`) passed directly into the code under test often replaces both the mock and the trait it mocks — no framework needed, and the compiler still checks the signature matches. Reserve Mockito, if used at all, for the external-boundary case in `mocking-discipline-mock-external-boundaries-only`, where the wrapped type is typically a plain (non-final, non-case) Java or Scala class from a third-party SDK that Mockito can subclass cleanly.

## Bad Example
```scala
test("confirms an order") {
  val repo = mock[OrderRepository[IO]]
  when(repo.findById(orderId)).thenReturn(IO.pure(Some(pendingOrder)))
  when(repo.save(any[Order])).thenReturn(IO.unit)

  OrderService(repo).confirm(orderId).unsafeRunSync()
  // the stub silently assumes findById never fails and save never fails,
  // which the real IO-returning implementation isn't guaranteed to uphold
}
```

## Good Example
```scala
class OrderServiceSuite extends munit.CatsEffectSuite:
  test("confirms an order") {
    val findById: OrderId => IO[Option[Order]] = _ => IO.pure(Some(pendingOrder))

    for
      savedRef <- Ref.of[IO, Option[Order]](None)
      save      = (order: Order) => savedRef.set(Some(order))
      _        <- OrderService(findById, save).confirm(orderId)
      saved    <- savedRef.get
    yield assertEquals(saved.map(_.status), Some(OrderStatus.Confirmed))
  }
```

## Notes
- Designing collaborators as plain functions instead of single-method traits, where the trait adds no other value, sidesteps the mocking question entirely.
- `mocking-discipline-in-memory-interpreters` is the preferred alternative for multi-method traits where a function value doesn't fit.
- When a function-value collaborator needs to record what it was called with, `Ref[IO, A]` gives that without a mutable `var` — the same discipline `mocking-discipline-in-memory-interpreters` applies at the scale of a full interpreter.

## References
- [Mockito — Scala limitations with final classes](https://github.com/mockito/mockito-scala#mocking-final-classes-and-methods)
