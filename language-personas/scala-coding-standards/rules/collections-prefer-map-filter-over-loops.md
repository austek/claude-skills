---
title: Prefer map/filter/fold Over Hand-Written Loops
impact: MEDIUM
impactDescription: removes the var accumulator and off-by-one index errors loops are prone to
tags: [collections, functional-style, immutability]
---

# Prefer map/filter/fold Over Hand-Written Loops [MEDIUM]

## Description
A `while`/`for` loop that mutates a `var` accumulator to build a result — summing values, collecting matches, building a lookup table — encodes *how* to iterate (initialize a variable, loop, check a condition, mutate, repeat) instead of *what* the result is. `map`, `filter`, `fold`/`foldLeft`, `collect`, and `groupBy` name the intent directly: `map` says "transform every element," `filter` says "keep the ones matching a predicate," `foldLeft` says "combine every element into one value starting from a seed." Because each of these methods owns its own iteration and termination logic, using them also removes an entire category of loop bugs — off-by-one bounds, forgetting to reset an accumulator, mutating the wrong variable inside a nested loop — since there's no index or mutable accumulator left for the caller to get wrong.

This is also where the house rule against `var` (`immutability-val-over-var`) bites in practice: a `var` almost always exists because something is being accumulated across iterations, and the standard collection operations are precisely the tool that accumulation pattern should be delegated to instead. `foldLeft` in particular subsumes a hand-written accumulation loop directly — its second argument *is* the accumulator, threaded through immutably one element at a time, with no `var` anywhere in the call.

A loop is still the right tool when the operation is inherently about a side effect with no result to build — `foreach` (or a genuine `while`) for writing to a stream or driving I/O per element — since routing a side-effecting action through `map` just to discard its result is not an improvement.

## Bad Example
```scala
def totalActive(orders: List[Order]): BigDecimal =
  var total = BigDecimal(0)
  for order <- orders do
    if order.status == OrderStatus.Active then
      total += order.amount
  total
```

## Good Example
```scala
def totalActive(orders: List[Order]): BigDecimal =
  orders
    .filter(_.status == OrderStatus.Active)
    .foldLeft(BigDecimal(0))(_ + _.amount)
```

## Notes
- `foldLeft(seed)(combine)` is the direct functional replacement for a `var` accumulator threaded through a loop — the seed is the initial value, and `combine` is the loop body's mutation, made explicit and immutable.
- `.collect(partialFunction)` fuses a filter and a map into one pass when the transformation is only defined for some elements, avoiding a separate `.filter` call followed by `.map`.
- `foreach` is the correct choice, not an exception to this rule, when the goal is a side effect per element with no collection result to build.
- `immutability-val-over-var` covers the general case against `var`; this rule covers the specific, common place a `var` shows up — a hand-rolled loop accumulator — and its direct fix.

## References
- [Scala Collections — Overview](https://docs.scala-lang.org/overviews/collections-2.13/overview.html)
