---
title: Never Use asInstanceOf/isInstanceOf; Pattern Match Instead
impact: HIGH
impactDescription: turns an unchecked runtime cast that can throw ClassCastException into a compiler-verified, exhaustive branch
tags: [anti-patterns, casting, pattern-matching, type-safety]
---

# Never Use asInstanceOf/isInstanceOf; Pattern Match Instead [HIGH]

## Description
`isInstanceOf[T]` followed by `asInstanceOf[T]` is a manual, unchecked re-implementation of what pattern matching already does safely: `isInstanceOf` answers "is this actually a `T` at runtime," and `asInstanceOf` then asserts it, with the compiler trusting the assertion completely and inserting no check of its own. Nothing connects the two calls — a reader (or the compiler) can't verify that the `isInstanceOf` check on one line actually guards the `asInstanceOf` cast on the next, and the two are trivially easy to get out of sync during a refactor, at which point the cast throws a `ClassCastException` with no compile-time warning that it might. This pair also loses the same information a `match` would give for free: no automatic narrowing of the value's type in the branch, no exhaustiveness check against a sealed hierarchy, and no compiler error if a new case is added to the hierarchy later and this code forgets to handle it.

Pattern matching against a sealed hierarchy (`pattern-matching-sealed-trait-exhaustiveness`) replaces both calls with one construct that the compiler actually checks: each `case` narrows the matched value to the pattern's type automatically, with no cast needed to use it, and the compiler flags a non-exhaustive match at the definition site rather than letting an unhandled case surface as a runtime exception somewhere else entirely. The only place a cast-like operation is occasionally unavoidable is at a genuine boundary with untyped or reflection-based Java APIs — even there, prefer a `match` against the type first when possible, since it gives the same safety with none of the two-call ceremony.

## Bad Example
```scala
def area(shape: Shape): Double =
  if shape.isInstanceOf[Circle] then
    val c = shape.asInstanceOf[Circle]
    math.Pi * c.radius * c.radius
  else if shape.isInstanceOf[Square] then
    shape.asInstanceOf[Square].side * shape.asInstanceOf[Square].side
  else 0.0
```

## Good Example
```scala
enum Shape:
  case Circle(radius: Double)
  case Square(side: Double)

def area(shape: Shape): Double =
  shape match
    case Shape.Circle(radius) => math.Pi * radius * radius
    case Shape.Square(side)   => side * side
```

## Notes
- `pattern-matching-sealed-trait-exhaustiveness` covers the compile-time exhaustiveness guarantee this rewrite depends on — it requires the matched hierarchy to actually be sealed (or a Scala 3 `enum`) for the compiler to check every case is handled.
- `pattern-matching-destructuring-over-accessors` covers the parallel benefit of extracting a case's fields directly in the pattern (`case Shape.Circle(radius) =>`) instead of a separate accessor call after the match.
- A `match` with a typed pattern (`case c: Circle =>`) against a *non-sealed* type still avoids the redundant two-call ceremony and gives automatic narrowing, even without the compiler's exhaustiveness check.
- `anti-patterns-no-null-return`'s empty-collection guidance and this rule share a theme: prefer a construct the type system verifies over one where safety depends on the programmer getting an unchecked, easy-to-desync pair of operations right every time.

## References
- [Scala 3 Book — Pattern Matching](https://docs.scala-lang.org/scala3/book/control-structures.html#match-expressions)
