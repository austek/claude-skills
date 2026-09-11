---
title: A Collection-Returning Method Returns Empty, Never null
impact: HIGH
impactDescription: removes the one extra null-check every caller of a collection-returning method would otherwise need
tags: [anti-patterns, null-safety, collections]
---

# A Collection-Returning Method Returns Empty, Never null [HIGH]

## Description
`null-option-no-null-literal` bans `null` as a value everywhere; this rule calls out the specific, easy-to-miss case where a method's own return type is already a collection. `List[LineItem]` already has a value that means "nothing here" — the empty list — so returning `null` instead adds a second, type-invisible way to signal the same thing, and it's strictly worse than the one the type already provides: an empty `List` supports every operation a non-empty one does (`.isEmpty`, `.map`, `.foreach`, folding) with no special-casing, while a `null` crashes the instant any of those is called without a guard the type system never asked for. A single `null`-returning method is also disproportionately damaging in a codebase that otherwise follows the no-`null` discipline everywhere else — every caller of *that one method* has to remember, from documentation or experience, that this particular collection might be `null`, breaking the blanket trust the rest of the codebase can place in a `List[T]` never needing a null-check.

Wrapping the return type in `Option[List[T]]` doesn't fix this either — it just relocates the same redundancy one level up, forcing every caller to unwrap an `Option` before they can even ask whether the list has anything in it, when the empty list already answers that question directly. Reach for `Option` around a collection only when "no collection was ever produced" is a genuinely distinct outcome from "an empty collection was produced" — a search that failed to run at all versus one that ran and matched nothing — not as a default wrapper for absence.

## Bad Example
```scala
class OrderRepository:
  def findLineItems(orderId: OrderId): List[LineItem] =
    if orderCache.contains(orderId) then orderCache(orderId).items
    else null // no items found — every caller must now guard against null before using this List at all
```

## Good Example
```scala
class OrderRepository:
  def findLineItems(orderId: OrderId): List[LineItem] =
    orderCache.get(orderId).fold(List.empty[LineItem])(_.items)
```

## Notes
- `null-option-no-null-literal` covers the general prohibition on `null` as a value anywhere; this rule is the specific, common case of a collection return type, where an empty collection is always available as the correct non-`null` answer.
- This applies the same way to `Map`, `Set`, `Vector`, or any other collection type — the empty instance (`Map.empty`, `Set.empty`) is always a valid, safe substitute for `null`.
- A Java interop boundary returning a possibly-`null` collection should be wrapped immediately (`Option(javaCall()).getOrElse(List.empty)` or an equivalent), the same "wrap at the boundary" discipline `null-option-no-null-literal` already prescribes for any nullable Java return value.
- Don't reach for `Option[List[T]]` by default — it adds an unwrap step for information the empty list already carries; reserve it for when "never ran" and "ran, found nothing" are genuinely different outcomes worth distinguishing.

## References
- [Scala 3 Reference — Explicit Nulls](https://docs.scala-lang.org/scala3/reference/other-new-features/explicit-nulls.html)
