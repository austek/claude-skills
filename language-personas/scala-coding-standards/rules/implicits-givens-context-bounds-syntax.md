---
title: Use Context Bound Syntax for a Single using Type Class Parameter
impact: LOW
impactDescription: states "requires evidence of T" in the type parameter list instead of a second, often-unused parameter clause
tags: [implicits, context-bounds, given]
---

# Use Context Bound Syntax for a Single using Type Class Parameter [LOW]

## Description
A generic function that requires a type class instance for its type parameter — `def sortedDescriptions[T](values: List[T])(using ord: Ordering[T]): List[String]` — can express that requirement more tersely with a context bound: `def sortedDescriptions[T: Ordering](values: List[T]): List[String]`. The `: Ordering` attached to `T` desugars to exactly the same anonymous `using` clause, added by the compiler; the two forms are interchangeable at the call site and identical after compilation. The context bound form reads as "T, for which an Ordering exists" directly in the type parameter list, which keeps a method's signature shorter whenever the instance is never referenced by name inside the body — only ever picked up implicitly by another call that needs it, such as `List#sorted`'s own implicit `Ordering` parameter.

The named `using` form is still the right choice, not a fallback, whenever the body actually calls a method on the instance by name — a context bound gives no name to reference, so recovering one requires `summon[Ordering[T]]`, which is more to read than simply naming the parameter with `using ord: Ordering[T]` in the first place. A context bound also only covers a single type class requirement per bound; a type parameter needing evidence from two type classes stacks bounds (`def describe[T: Ordering: Show](value: T): String`), still more compact than spelling out two named `using` parameters neither of which is referenced directly.

## Bad Example
```scala
def sortedDescriptions[T](values: List[T])(using ord: Ordering[T]): List[String] =
  values.sorted.map(_.toString) // ord is picked up implicitly by sorted; the name itself is never used
```

## Good Example
```scala
def sortedDescriptions[T: Ordering](values: List[T]): List[String] =
  values.sorted.map(_.toString)
```

## Notes
- `summon[Ordering[T]]` retrieves the instance a context bound brought into scope without naming it in the parameter list — the standard way to reach it when the body needs to call a method on it directly.
- Once the instance needs a name for repeated, readable use in the body, switch back to a named `using ord: Ordering[T]` clause instead of calling `summon` repeatedly.
- Multiple context bounds on one type parameter (`T: Ordering: Show`) stack in the parameter list; each still desugars to its own `using` parameter.
- `implicits-givens-type-class-pattern` covers the broader pattern this syntax is sugar for.

## References
- [Scala 3 Reference — Context Bounds](https://docs.scala-lang.org/scala3/reference/contextual/context-bounds.html)
