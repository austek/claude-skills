---
title: Model Capabilities as Type Classes With given Instances
impact: HIGH
impactDescription: adds behavior to a type without touching its definition or reaching for inheritance
tags: [implicits, type-class, given]
---

# Model Capabilities as Type Classes With given Instances [HIGH]

## Description
A type class separates *what a type can do* from *the type's own definition*: a trait (`JsonEncoder[T]`, `Ordering[T]`, `Show[T]`) declares an operation parameterized by `T`, a `given` instance implements that operation for one concrete type, and a `using` clause on a generic function requires the caller's context to provide such an instance for whatever `T` it's called with. This solves a problem inheritance can't: adding a capability to a type without touching its source and without every value of that type being forced to carry every capability anyone might ever want. A third-party `User` class, or a type from the standard library, can get a `JsonEncoder[User]` instance defined anywhere in the codebase with no change to `User` itself — compare that to baking a single `.toJson` method directly onto the class, which commits every consumer to one encoding format whether they need it or not.

A `using` clause on a generic function is what makes the capability available polymorphically: `def render[T](value: T)(using encoder: JsonEncoder[T]): String` works for any `T` that has a `given JsonEncoder[T]` in scope, resolved at compile time with no runtime dispatch overhead and no possibility of a missing instance surfacing as anything other than a compile error. This is the mechanism `Ordering`, `Numeric`, and third-party patterns like Cats' `Show`/`Monoid` are all built on: define the capability once as a trait, provide instances per type via `given`, and require the capability generically via `using`.

## Bad Example
```scala
trait JsonEncoder[T]:
  def encode(value: T): String

class UserJsonEncoder extends JsonEncoder[User]:
  def encode(value: User): String = s"""{"name":"${value.name}"}"""

def render(value: User, encoder: JsonEncoder[User]): String =
  encoder.encode(value) // caller must thread the right encoder through by hand at every call site
```

## Good Example
```scala
trait JsonEncoder[T]:
  def encode(value: T): String

given JsonEncoder[User] with
  def encode(value: User): String = s"""{"name":"${value.name}"}"""

def render[T](value: T)(using encoder: JsonEncoder[T]): String =
  encoder.encode(value) // resolved from scope for whatever T the caller passes, no manual threading
```

## Notes
- Instances resolve at compile time via implicit search; a missing `given` for the required type is a compile error, not a runtime `NoSuchElementException` or null.
- This is the same shape `Ordering`, `Numeric`, and third-party type classes use — recognizing trait + given instances + using clause transfers directly to reading library code.
- `implicits-givens-context-bounds-syntax` covers the `[T: JsonEncoder]` shorthand for the `using` clause this pattern relies on.
- `implicits-givens-extension-methods-over-implicit-class` covers adding ergonomic `value.toJson`-style call syntax on top of a type class instance, once one exists.

## References
- [Scala 3 Book — Type Classes](https://docs.scala-lang.org/scala3/book/ca-type-classes.html)
