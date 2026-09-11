---
title: Model Closed Hierarchies with sealed trait/enum for Exhaustive match
impact: CRITICAL
impactDescription: turns a missed variant into a compile error at every match site that needs updating
tags: [pattern-matching, sealed-trait, enum, exhaustiveness]
---

# Model Closed Hierarchies with sealed trait/enum for Exhaustive match [CRITICAL]

## Description
A `sealed trait` (or a Scala 3 `enum`, which compiles to a sealed hierarchy under the hood) restricts all of a type's direct subtypes to the same source file — the compiler therefore knows the complete set of variants and can check whether a `match` over that type covers every one of them. This is the entire payoff of sealing a hierarchy: the compiler emits a "match may not be exhaustive" warning (promotable to a build-breaking error with `-Xfatal-warnings` or `-Werror`) the moment a `match` doesn't handle every variant, and — more importantly — the moment a new variant is added to an already-sealed hierarchy, every existing `match` that doesn't yet handle it starts failing to compile, at exactly the call sites that need updating. An *unsealed* trait, or matching on `Any`/a broad supertype, gets none of this: the compiler cannot know the full variant set, so it cannot check completeness, and a missed case only surfaces as a runtime `MatchError`.

Prefer Scala 3's `enum` for a closed set of variants with uniform, simple construction (`enum Color { case Red, Green, Blue }`); fall back to `sealed trait` with case classes/objects when variants need to carry heterogeneous fields, extend additional traits, or otherwise don't fit an `enum`'s single-hierarchy shape. Either way, the modeling choice and the exhaustiveness check it buys are inseparable from how errors and domain results should be represented — see `error-handling-typed-errors-over-strings` for using exactly this shape as an error channel.

## Bad Example
```scala
trait Shape // not sealed — the compiler cannot know every subtype
class Circle(val radius: Double) extends Shape
class Rectangle(val width: Double, val height: Double) extends Shape

def area(shape: Shape): Double = shape match
  case c: Circle    => math.Pi * c.radius * c.radius
  case r: Rectangle => r.width * r.height
  // a new Shape subtype added anywhere in the codebase compiles silently here
  // and throws MatchError at runtime the first time it reaches this match
```

## Good Example
```scala
enum Shape:
  case Circle(radius: Double)
  case Rectangle(width: Double, height: Double)
  case Triangle(base: Double, height: Double)

def area(shape: Shape): Double = shape match
  case Shape.Circle(r)       => math.Pi * r * r
  case Shape.Rectangle(w, h) => w * h
  case Shape.Triangle(b, h)  => 0.5 * b * h
  // no default — compiler verifies this covers every Shape case
  // adding a new case breaks this match until it is handled here
```

## Notes
- Enable `-Xfatal-warnings` (or the equivalent `-Werror`) in build settings so a non-exhaustive match fails the build instead of only printing a warning that can be missed in CI output.
- Exhaustiveness checking requires matching on the sealed type itself (or an interface whose every permitted subtype is covered) — matching on `Any` or an unrelated supertype forfeits the guarantee even if every concrete subtype happens to be sealed.
- `pattern-matching-no-catch-all-unless-intentional` covers the specific way this guarantee gets silently defeated even after correctly sealing a hierarchy: adding a `case _ =>` wildcard.
- Scala 3's `enum` also supports parameterized cases and methods, making it strictly more capable than Java's enum for this role — it is the idiomatic default over hand-written `sealed trait` boilerplate for the common case.

## References
- [Scala 3 Reference — Enums](https://docs.scala-lang.org/scala3/reference/enums/enums.html)
- [Scala 3 Reference — Sealed Traits](https://docs.scala-lang.org/scala3/reference/other-new-features/open-classes.html)
