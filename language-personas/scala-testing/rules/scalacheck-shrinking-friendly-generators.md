---
title: Keep Generators Shrinking-Friendly
impact: MEDIUM
impactDescription: turns a 200-element failing counterexample into a 2-element one, cutting failure-diagnosis time
tags: [scalacheck, generators, shrinking]
---

# Keep Generators Shrinking-Friendly [MEDIUM]

## Description
When a property fails, ScalaCheck doesn't just report the first failing input — it shrinks it, repeatedly trying smaller variants (`Shrink[A]`) until it finds a minimal case that still fails, then reports that. This only works well if the generator's structure lines up with the type's own `Shrink` instance: built-in types (`Int`, `String`, `List[A]`, `Option[A]`) already shrink toward zero/empty/`None`, and a generator composed from them (`scalacheck-generator-composition`) inherits that behavior automatically because ScalaCheck derives `Shrink` for case classes and tuples from their fields' `Shrink` instances.

Shrinking breaks down for generators that build a value and then discard structure the `Shrink` instance would need — for example wrapping a generated `String` in a hand-rolled type with no derived `Shrink`, or generating via `Gen.suchThat` on a broad type, where a shrunk candidate is likely to fail the predicate and get skipped. For a domain type with real invariants, define its own `given Shrink[A]` that only ever produces valid smaller values, rather than leaving shrinking to produce nothing (or worse, an invalid value that gets silently discarded, hiding a smaller counterexample).

## Bad Example
```scala
final case class OrderId(value: String) // no Shrink instance derivable — String's Shrink doesn't apply here
val genOrderId: Gen[OrderId] =
  Gen.identifier.suchThat(_.length >= 4).map(OrderId.apply)
// a failure shrinks the identifier but then filters out most shrunk candidates via suchThat, so
// ScalaCheck reports whatever it found first rather than a minimal case
```

## Good Example
```scala
final case class OrderId(value: String)

val genOrderId: Gen[OrderId] =
  Gen.choose(4, 12).flatMap(n => Gen.stringOfN(n, Gen.alphaNumChar)).map(OrderId.apply)

given Shrink[OrderId] =
  Shrink.shrinkString.map(OrderId.apply).suchThat(_.value.length >= 4)
```

## Notes
- `Shrink.shrinkAny[A]` (no shrinking) is a legitimate, explicit choice for a type where any shrunk value would be meaningless — better than an instance that silently produces nothing useful.
- Shrinking runs *after* a failure is found; it never affects which inputs get generated in the first place, only how the reported counterexample is minimized.
- A generator that composes only built-in generators (`scalacheck-generator-composition`) rarely needs a custom `Shrink` at all — the derived one is usually sufficient.

## References
- [ScalaCheck — User Guide, Shrinking](https://github.com/typelevel/scalacheck/blob/main/doc/UserGuide.md#test-case-minimisation)
