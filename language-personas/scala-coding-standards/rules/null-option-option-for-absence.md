---
title: Represent Optional Values with Option[T]
impact: HIGH
impactDescription: makes absence a compiler-checked branch instead of an undocumented runtime possibility
tags: [option, nullability, api-design]
---

# Represent Optional Values with Option[T] [HIGH]

## Description
When a value may legitimately be absent — a lookup that might not find anything, a configuration key that might not be set, a field that doesn't apply to every instance — put that in the type signature as `Option[T]` rather than returning `T` and relying on a sentinel, a `null`, or documentation to communicate that absence is possible. `Option[T]` is `Some(value)` or `None`; the compiler will not let code treat an `Option[T]` as a plain `T`, so every caller has to explicitly deal with the absent case, via pattern matching, `map`/`flatMap`, `fold`, `getOrElse`, or a `for`-comprehension. That requirement is the entire value of the type: absence becomes a compile-time fact about the API, not a footnote a caller has to remember to check.

`Option` composes the way any monadic type does: `map` transforms the value if present and is a no-op on `None`; `flatMap` chains together multiple operations that each might themselves produce `None`, short-circuiting on the first one; a `for`-comprehension over several `Option`s reads as straight-line code while still short-circuiting correctly underneath. This composability is what makes `Option` preferable to a boolean "found" flag plus a separately-fetched value, or to a pair of "hasX"/"getX" methods — those patterns can be called out of order or inconsistently, where `Option` structurally forces the check-then-use sequence.

## Bad Example
```scala
def findDiscount(code: String): BigDecimal =
  discounts.getOrElse(code, BigDecimal(0)) // silently means "no discount", indistinguishable from a real 0

def hasDiscount(code: String): Boolean = discounts.contains(code)
def getDiscount(code: String): BigDecimal = discounts(code) // throws if called without checking first
```

## Good Example
```scala
def findDiscount(code: String): Option[BigDecimal] =
  discounts.get(code)

val total = for
  discount <- findDiscount(code)
  price    <- findPrice(sku)
yield price - discount

val display = findDiscount(code).fold("no discount")(d => s"$d off")
```

## Notes
- `Option` is a "value or nothing" channel — it carries no information about *why* something is absent; when the reason matters to the caller, use `Either[E, A]` instead (`null-option-either-for-expected-failure`).
- Avoid `Option[Option[T]]`; if an operation can produce that shape (e.g. `map` over an `Option`-returning function), `flatMap` or `.flatten` it rather than threading the nesting further.
- Returning `Option[List[T]]` to mean "no results" is usually wrong — return a plain, possibly-empty `List[T]` instead, and reserve `Option` for cases where "no list at all" is meaningfully different from "an empty list."
- `null-option-avoid-option-get` and `null-option-orelse-chains-over-nested-match` cover the two most common ways teams accidentally defeat `Option`'s safety after adopting it.

## References
- [Scala 3 Book — Working with Option](https://docs.scala-lang.org/scala3/book/first-look-at-types.html#the-option-type)
