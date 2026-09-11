---
title: Default to Structure-Sharing Immutable Collections
impact: MEDIUM
impactDescription: safe to hash, use as a Map key or Set element, and share without defensive copying
tags: [collections, immutability, persistent-data-structures]
---

# Default to Structure-Sharing Immutable Collections [MEDIUM]

## Description
Scala's immutable collections (`List`, `Vector`, `Map`, `Set`) are persistent data structures: an "update" like `xs :+ elem` or `map + (k -> v)` returns a new collection that shares most of its internal structure with the original instead of copying it wholesale, so producing a new version from an old one is cheap — amortized O(1) for `List` prepend, effectively O(1) for `Vector` append, O(log n) for `Map`/`Set` insert — rather than a full O(n) copy. That structural sharing is what makes "default to immutable" a genuine engineering default rather than a purity tax: choosing `List`/`Vector`/`Map`/`Set` over their mutable counterparts is not choosing to pay a copying cost every time data changes, it's choosing a representation designed to make change cheap while never letting one reference's write become visible through another's.

The default also matters for correctness beyond thread-safety: `equals`/`hashCode` on an immutable collection are structural and stable for the object's lifetime, so it can safely be used as a `Map` key or a `Set` element. A mutable collection's `hashCode` is also structural — but nothing stops it from changing after insertion into a hash-based structure, which silently corrupts that structure's invariants (the entry becomes unreachable at its own bucket) without raising an error. This is a sharper failure than the general API-boundary leak `immutability-immutable-collections-default` describes: it is not a caller mutating shared state, it is a data structure invariant breaking because a key's hash changed underneath it.

Reach for `Vector` as the general-purpose sequence, `List` specifically for head/tail recursion, and `Map`/`Set` for associative and membership lookups; reserve a mutable collection for a local accumulator that never leaves its constructing scope (`collections-avoid-mutable-builder-escape`).

## Bad Example
```scala
import scala.collection.mutable
import scala.collection.immutable.HashSet

val tag = mutable.Set("urgent")
val seenTags: HashSet[mutable.Set[String]] = HashSet(tag)

tag += "reviewed" // mutates tag after insertion — its bucket in seenTags is now stale
seenTags.contains(tag) // false: hashCode changed since insertion, lookup misses the same reference
```

## Good Example
```scala
import scala.collection.immutable.HashSet

val tag: Set[String] = Set("urgent")
val seenTags: HashSet[Set[String]] = HashSet(tag)

val updatedTag = tag + "reviewed" // a new, independent Set — tag itself never changes
seenTags.contains(tag) // true: tag's hashCode never changed after insertion
```

## Notes
- Persistent data structures share structure between versions, so an "update" on `List`/`Vector`/`Map`/`Set` is not a literal full copy — this is what makes immutable-by-default cheap, not only safe.
- Never use a mutable collection as a `Map` key or `Set` element; its `hashCode` can change after insertion and silently break the hash table's invariants.
- `immutability-immutable-collections-default` covers the general API-boundary case for preferring immutable over mutable; this rule covers the structural-sharing and hashing reasons immutable is the right *default*, not only the boundary-safety case.
- `Vector` is the general-purpose immutable sequence; `List` when the access pattern is genuinely head/tail recursion.

## References
- [Scala Collections — Performance Characteristics](https://docs.scala-lang.org/overviews/collections-2.13/performance-characteristics.html)
