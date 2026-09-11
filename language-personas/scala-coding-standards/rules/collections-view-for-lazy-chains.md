---
title: Use .view to Avoid Materializing Intermediate Collections in a Chain
impact: MEDIUM
impactDescription: turns an N-allocation transformation chain into one, by deferring every step until a single terminal call
tags: [collections, view, laziness]
---

# Use .view to Avoid Materializing Intermediate Collections in a Chain [MEDIUM]

## Description
Chaining several transformations on a strict collection — `.filter(...).map(...).take(n)` on a `List` or `Vector` — allocates a full intermediate collection after every single step, even though only the final result is ever used; a four-call chain over a large `Vector` builds several large intermediate collections just to discard all but one of them. `.view` converts a collection into a lazy, non-strict wrapper: each transformation on a view returns another view describing the operation rather than running it, and nothing executes until a terminal, strict operation — `.toList`, `.toVector`, `.sum`, `foreach` — is called, at which point every queued transformation runs in a single pass with no intermediate collection materialized at all.

This makes `.view` the right tool specifically for a chain of several intermediate operations (`filter`, `map`, `collect`, `flatMap`) over a collection that is already fully in memory and finite — for a genuinely unbounded or self-referential sequence, see `collections-lazylist-for-infinite-sequences` instead. `.view` pairs especially well with `.take`, since the fused pipeline can then stop after the first N matching elements instead of scanning the whole source through every step. It is not a general performance switch to sprinkle onto every collection operation: a view has per-element overhead from the layered wrapped closures, so a single `.map` call or a chain over a small collection is often faster left strict — and converting a view back to a strict collection re-runs the entire queued chain, so forcing the same view more than once repeats all of the work rather than reusing a cached result.

## Bad Example
```scala
def topActiveEmails(users: Vector[User]): List[String] =
  users
    .filter(_.isActive) // allocates a full intermediate Vector
    .map(_.email)       // allocates another full intermediate Vector
    .take(10)
    .toList
```

## Good Example
```scala
def topActiveEmails(users: Vector[User]): List[String] =
  users.view
    .filter(_.isActive) // queues the filter; allocates nothing yet
    .map(_.email)        // queues the map; allocates nothing yet
    .take(10)             // queues the bound; the pipeline stops after the first 10 matches
    .toList                // the only strict step: runs filter + map + take in one pass
```

## Notes
- A view re-runs its whole transformation chain every time it's converted to a strict collection; store the strict `.toList`/`.toVector` result if the same data will be consumed more than once.
- `.view` over a mutable collection reflects later mutations to that collection, since the view holds a reference rather than a snapshot — force it immediately if the source might change before the view is consumed.
- Reach for `.view` when chaining several transformations, not for a single `.map` or `.filter` call, where the wrapping overhead has no matching allocation to save.
- `collections-lazylist-for-infinite-sequences` covers the related but distinct case of a lazily-generated, self-extending sequence, rather than a lazy wrapper over an already-materialized collection.

## References
- [Scala Collections — Views](https://docs.scala-lang.org/overviews/collections-2.13/views.html)
