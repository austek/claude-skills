---
title: Prefer Fakes Over Mocks
impact: HIGH
impactDescription: decouples tests from a collaborator's call sequence, so refactoring its internals doesn't break unrelated tests
tags: [mocking, fakes, test-doubles]
---

# Prefer Fakes Over Mocks [HIGH]

## Description
A fake is a real, working implementation of a trait built for tests — an in-memory repository backed by a `Map`, a clock that returns a fixed instant — that a test exercises through its actual interface and asserts on by checking observable state afterward. A mock is a recording stand-in that asserts on *how* it was called: exact arguments, call count, call order. Prefer fakes. A test built on a fake only breaks when the collaborator's observable behavior changes; a test built on a mock also breaks when its *implementation* changes in some way that alters call sequence, even if the observable outcome is identical — for example switching `save` then `publish` to `publish` then `save` with no behavior difference a caller could detect.

Mocks earn their keep for one thing fakes can't easily provide: asserting that a specific side effect *happened*, when there's no state to inspect afterward (an email was sent, an audit log call fired). Reach for a fake by default and fall back to a mock only for that narrow case, not as the default test-double strategy.

## Bad Example
```scala
test("processing an order saves it") {
  val repo = mock[OrderRepository]
  when(repo.findById(orderId)).thenReturn(Some(pendingOrder))

  OrderService(repo).confirm(orderId)

  verify(repo).save(pendingOrder.copy(status = OrderStatus.Confirmed))
  // breaks if OrderService starts calling save() with an intermediate state first,
  // even though the final persisted order would still be correct
}
```

## Good Example
```scala
test("processing an order saves it") {
  val repo = InMemoryOrderRepository.withOrder(pendingOrder)

  OrderService(repo).confirm(orderId)

  assertEquals(repo.findById(orderId).map(_.status), Some(OrderStatus.Confirmed))
  // asserts only the observable outcome, independent of how many times save() ran
}
```

## Notes
- `mocking-discipline-in-memory-interpreters` covers writing the fake itself for repository- and client-shaped traits.
- A fake still needs its own tests (or must be trivial enough not to), or it can silently drift from the real implementation's contract and give false confidence.
- `mocking-discipline-mock-external-boundaries-only` narrows down exactly where a mock remains the right tool.

## References
- [Martin Fowler — Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html)
