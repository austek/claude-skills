---
title: Never Return a Collection Builder Before Calling .result()
impact: MEDIUM
impactDescription: prevents a shared Builder from being mutated after it's supposed to be finished
tags: [collections, builder, mutable-collections]
---

# Never Return a Collection Builder Before Calling .result() [MEDIUM]

## Description
Scala's `scala.collection.mutable.Builder[Elem, To]`, obtained from `List.newBuilder[A]`, `Vector.newBuilder[A]`, or any collection's `.newBuilder`, is the standard low-level tool for constructing a collection incrementally when a `.map`/`.filter` chain or a `for`-comprehension (`for-comprehensions-monadic-composition`) would be awkward or too slow — it's the same mechanism the standard library's own collection operations use internally. A `Builder` is mutable and single-use by contract: `.addOne`/`+=` accumulates elements, and `.result()` finalizes and hands back the immutable collection. Treating a `Builder` like an ordinary local value instead of respecting that contract causes two distinct mistakes: returning or sharing the `Builder` itself instead of its `.result()`, which lets an unrelated caller keep adding elements to what looks like a finished collection; and calling `.result()` more than once, or calling `.addOne` again afterward. The `Builder` contract makes no guarantee about what happens in that second case — some implementations reject it outright, others silently allow further mutation that leaves a previously returned collection referencing now-invalidated internal state — so both are bugs regardless of which a given implementation happens to do today.

The fix mirrors the general accumulator-escape discipline (`immutability-avoid-mutable-builder-escape`) but applies it specifically to this API: keep the `Builder` entirely local to the method that creates it, call `.result()` exactly once, and return only that result — never the `Builder` reference, and never touch it again afterward.

## Bad Example
```scala
import scala.collection.mutable

class BatchAccumulator:
  private val builder = List.newBuilder[Int]
  def add(n: Int): Unit = builder += n
  def pending: mutable.Builder[Int, List[Int]] = builder // leaks the live, unfinished builder
```

## Good Example
```scala
class BatchAccumulator:
  private val builder = List.newBuilder[Int]
  def add(n: Int): Unit = builder += n
  def build(): List[Int] = builder.result() // the only place result() is called; builder never escapes
```

## Notes
- A `Builder`'s `.result()` contract guarantees nothing about the builder's state afterward — treat every `Builder` as single-use, finalized from exactly one call site.
- `immutability-avoid-mutable-builder-escape` covers the general case of any mutable accumulator (`ArrayBuffer`, `StringBuilder`, `mutable.Map`) escaping its constructing scope; this rule applies the same discipline specifically to the stdlib's `Builder` type.
- Most call sites don't need `.newBuilder` directly — `.map`/`.filter`/a `for`-comprehension covers the common case of building a collection without touching `Builder` at all.
- `sizeHint(n)` on a `Builder` preallocates capacity when the final size is known ahead of time, avoiding intermediate resizes — a performance detail, not a correctness one.

## References
- [Scala Standard Library — Builder](https://www.scala-lang.org/api/current/scala/collection/mutable/Builder.html)
