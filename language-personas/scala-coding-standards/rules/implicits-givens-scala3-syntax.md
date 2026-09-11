---
title: Use given/using, Not Scala 2 implicit, for New Code
impact: MEDIUM
impactDescription: separates "providing a capability" (given) from "requiring one" (using) into two keywords instead of one overloaded implicit
tags: [implicits, given, scala3-syntax]
---

# Use given/using, Not Scala 2 implicit, for New Code [MEDIUM]

## Description
Scala 2 overloads a single keyword, `implicit`, for several distinct purposes: declaring an implicit value, declaring an implicit parameter, declaring an implicit conversion, and providing type class evidence. Scala 3 splits this into purpose-specific keywords: `given` declares an instance of a type available for implicit search — "here is a capability" — and `using` marks a parameter that should be filled in from that search rather than from an explicit call-site argument — "this method requires a capability." The syntax also removes boilerplate for the common type class case: `given Ordering[Person] with { ... }` never requires naming the instance, whereas the Scala 2 equivalent (`implicit val personOrdering: Ordering[Person] = new Ordering[Person] { ... }`) requires inventing a name purely to satisfy the compiler, even when nothing ever refers to it directly.

For new Scala 3 code, prefer `given`/`using` over the legacy `implicit` keyword in every one of these roles. It makes the two directions — providing evidence versus requiring it — visually distinguishable at both the definition site and the call site, since a `using` clause in a signature is unmistakable in a way an `implicit` parameter clause is not (it's also implicitly marking the parameter list as implicit, an easy detail to miss when scanning a signature). It's also the form every later Scala 3 given-based feature — context bounds, extension methods requiring evidence — is built around, so code already using `given`/`using` composes with those features without a syntax mismatch.

## Bad Example
```scala
trait Ordering[T]:
  def compare(x: T, y: T): Int

implicit val intOrdering: Ordering[Int] = new Ordering[Int]:
  def compare(x: Int, y: Int): Int = x.compareTo(y)

def max[T](x: T, y: T)(implicit ord: Ordering[T]): T =
  if ord.compare(x, y) >= 0 then x else y
```

## Good Example
```scala
trait Ordering[T]:
  def compare(x: T, y: T): Int

given Ordering[Int] with
  def compare(x: Int, y: Int): Int = x.compareTo(y)

def max[T](x: T, y: T)(using ord: Ordering[T]): T =
  if ord.compare(x, y) >= 0 then x else y
```

## Notes
- `given T with { ... }` and `given T = expr` are the two given forms: `with` implements a trait/class body, `= expr` supplies an already-constructed value directly.
- A `using` clause can name its parameter (`using ord: Ordering[T]`) when the body needs to reference it directly, or omit the name (`using Ordering[T]`) when it's only ever passed along to another `using` clause.
- `implicits-givens-context-bounds-syntax` covers the further sugar for a single-type `using` clause on a type parameter.
- `given`/`using` and legacy `implicit` participate in the same implicit search and can coexist during a migration, but new code should use `given`/`using` exclusively.

## References
- [Scala 3 Reference — Given Instances](https://docs.scala-lang.org/scala3/reference/contextual/givens.html)
