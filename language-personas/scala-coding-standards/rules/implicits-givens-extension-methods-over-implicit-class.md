---
title: Use extension Methods, Not implicit class, to Add Methods to an Existing Type
impact: MEDIUM
impactDescription: adds a method with no allocation-per-call wrapper, and no AnyVal restrictions to satisfy
tags: [implicits, extension-methods, scala3-syntax]
---

# Use extension Methods, Not implicit class, to Add Methods to an Existing Type [MEDIUM]

## Description
Scala 2's idiom for adding a method to a type you don't control is an `implicit class` wrapping the target type: `implicit class Ops(value: T) { def newMethod = ... }`, which the compiler rewrites into a real class plus an implicit conversion function. Calling `.newMethod` on a `T` allocates an instance of `Ops` to receive the call — avoidable only by additionally extending `AnyVal` and satisfying its restrictions (a single public constructor parameter, no other val/var fields), restrictions that are easy to violate by accident since the compiler doesn't always point clearly at which one was broken.

Scala 3's `extension` syntax expresses the same capability directly, with no wrapper class and no allocation at all: `extension (value: T) def newMethod: R = ...` declares a method that resolves the same way at the call site (`t.newMethod`) but compiles to a plain static call taking `value` as an ordinary parameter. It also reads more directly as what it is — "here is an additional method for `T`" — rather than as a class definition whose real purpose (enabling implicit method resolution) is a level of indirection away from what the syntax shows. Extension methods can take their own type and value parameters (`extension [T](xs: List[T]) def secondOption: Option[T] = ...`) and can require a `using` clause the same way an ordinary method can, which is how they compose with the type class pattern (`implicits-givens-type-class-pattern`) — a common combination is an extension method whose body delegates to a `given` instance.

## Bad Example
```scala
implicit class RichString(val value: String) extends AnyVal:
  def isBlankOrEmpty: Boolean = value.trim.isEmpty
```

## Good Example
```scala
extension (value: String)
  def isBlankOrEmpty: Boolean = value.trim.isEmpty
```

## Notes
- Extension methods never allocate a wrapper instance at the call site — there is no `AnyVal`-style restriction to satisfy in the first place.
- Multiple extension methods on the same type can share one `extension (value: T)` clause by indenting them underneath it, instead of each needing its own wrapper class.
- An extension method requiring a `using` clause (`extension [T](xs: List[T])(using ord: Ordering[T]) def sorted2: List[T] = xs.sorted`) is how the type class pattern gets ergonomic call syntax on top of a `given` instance.
- `implicit class` still compiles under Scala 3 for migration purposes, but new code should use `extension`.

## References
- [Scala 3 Book — Extension Methods](https://docs.scala-lang.org/scala3/book/ca-extension-methods.html)
