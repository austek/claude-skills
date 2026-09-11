---
title: Don't Force a For-Comprehension Where a Direct Call Reads Better
impact: LOW
impactDescription: keeps single-step and non-dependent code free of needless for/yield ceremony
tags: [for-comprehension, readability, code-style]
---

# Don't Force a For-Comprehension Where a Direct Call Reads Better [LOW]

## Description
A `for`-comprehension earns its keep by replacing several *dependent* monadic steps with a linear read (`for-comprehensions-avoid-nested-flatmap`). Applied reflexively to every optional or effectful value regardless of shape, it adds ceremony without a payoff: a `for`-comprehension with exactly one generator and a `yield` is always equivalent to a single `.map` call, so writing the four-line `for` form to avoid a one-line `.map` is strictly more to read for no gain. The same applies to filtering a collection — `.filter`/`.map`/`.collect` (`collections-prefer-map-filter-over-loops`) state the intent more directly than wrapping the same logic in `for ... if ... yield`.

There is also a sharper reason to avoid a stray guard clause inside a `for`-comprehension used as a substitute for a real conditional: a guard `if` desugars to a `withFilter` call on the preceding generator's value, and the exact behavior of that depends entirely on what `withFilter` does for that type — for `Option` it silently narrows a match to `None`, for `List` it silently narrows to the empty list, and for a type with no proper `withFilter` implementation, Scala falls back to `filter`, which can throw at runtime for a pattern-match generator that doesn't account for every case. None of this is a compile error, which makes it easy to reach for a guard clause where an explicit `if`/`else` or `.filter` call would make the fallback behavior visible instead of implicit.

Reserve `for`-comprehensions for composing two or more steps of the *same* effect type where a later step depends on an earlier one's result — that is where the sequential syntax pays for its own ceremony.

## Bad Example
```scala
def discountedPrice(order: Order): Option[Double] =
  for
    total <- order.total
  yield total * 0.9

def expensiveOrders(orders: List[Order]): List[Order] =
  for
    order <- orders
    if order.total.exists(_ > 1000)
  yield order
```

## Good Example
```scala
def discountedPrice(order: Order): Option[Double] =
  order.total.map(_ * 0.9)

def expensiveOrders(orders: List[Order]): List[Order] =
  orders.filter(order => order.total.exists(_ > 1000))
```

## Notes
- A `for`-comprehension with one generator and a `yield` is always exactly `generator.map(x => yieldExpr)` — if that is the entire block, write the `.map` call.
- A guard `if` inside a `for`-comprehension desugars to `.filter`/`.withFilter` on the preceding generator's type; know what that means for the specific type before relying on it instead of an explicit conditional.
- `for-comprehensions-avoid-nested-flatmap` covers the opposite failure — hand-nesting three or more dependent steps instead of using this sugar.
- This is a style boundary, not a type-specific rule — it applies to `Option`, `Either`, `List`, and effect types alike.

## References
- [Scala 3 Book — For Expressions](https://docs.scala-lang.org/scala3/book/control-structures.html#for-expressions)
