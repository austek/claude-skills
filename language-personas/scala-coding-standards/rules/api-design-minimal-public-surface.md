---
title: Keep the Public Surface to What Callers Actually Need
impact: MEDIUM
impactDescription: shrinks what a caller (and a future maintainer) has to read to understand how to use a type correctly
tags: [api-design, encapsulation, private, information-hiding]
---

# Keep the Public Surface to What Callers Actually Need [MEDIUM]

## Description
Every public member of a class is a promise: callers may depend on it, so it can't change or disappear without a coordinated update everywhere it's used. A method exposed only because it happens to be called from within the same class — a validation step, an intermediate calculation, a formatting helper — makes that same promise by accident, to no benefit: nothing outside the class was ever meant to call it, but now nothing stops external code from doing so, coupling to it, and turning an implementation detail into a de facto part of the contract. `private` (or the narrower `private[this]`) marks a member as exactly what it is — internal machinery, not part of the type's usable API — and the compiler enforces the boundary the same way it enforces any other type constraint.

This isn't about hiding for its own sake; it's about making the type's actual contract legible. A class where every field and helper method is public gives a reader no way to tell, just from the member list, which handful of methods are the ones meant to be called versus which are implementation steps of those methods — the reader has to trace the call graph to find out. Marking the internal steps `private` collapses that ambiguity: what's left public *is* the contract, and it can be read as a summary of what the type does without also reading how. Default new members to `private` and widen access deliberately, only when a concrete caller outside the type needs it — the reverse default, defaulting to public and narrowing later, requires noticing the narrowing was ever needed.

## Bad Example
```scala
class OrderProcessor:
  def process(order: Order): Either[OrderError, Receipt] =
    for
      validated <- validate(order)
      priced    <- price(validated)
    yield issueReceipt(priced)

  def validate(order: Order): Either[OrderError, Order] = ??? // public but only ever called from process
  def price(order: Order): Either[OrderError, PricedOrder] = ???
  def issueReceipt(priced: PricedOrder): Receipt = ???
```

## Good Example
```scala
class OrderProcessor:
  def process(order: Order): Either[OrderError, Receipt] =
    for
      validated <- validate(order)
      priced    <- price(validated)
    yield issueReceipt(priced)

  private def validate(order: Order): Either[OrderError, Order] = ???
  private def price(order: Order): Either[OrderError, PricedOrder] = ???
  private def issueReceipt(priced: PricedOrder): Receipt = ???
```

## Notes
- `private[this]` is stricter than `private` — it hides a member even from other instances of the same class, useful when a helper genuinely has no business being called on `other.helper`.
- A test that needs to reach a `private` method directly is usually a sign the method deserves its own top-level, independently testable unit, not a reason to widen the access modifier.
- This applies to case class fields the same way: a field only ever read through a computed accessor should not also be a public constructor parameter.
- `api-design-companion-object-factories` covers the parallel discipline for construction specifically — keeping the raw constructor out of the public surface entirely.

## References
- [Scala 3 Reference — Private Members](https://docs.scala-lang.org/scala3/reference/other-new-features/open-classes.html)
