---
title: Default to Immutable Collections
impact: HIGH
impactDescription: removes shared-mutable-state bugs and unsynchronized concurrent access as a category
tags: [immutability, collections, scala-collections]
---

# Default to Immutable Collections [HIGH]

## Description
Scala's unqualified collection names — `List`, `Vector`, `Map`, `Set` — resolve to `scala.collection.immutable` through the aliases in `scala.Predef`; reaching for `scala.collection.mutable` is an explicit opt-in, not the default. Keep it that way at every API boundary: function parameters and return types should be immutable collections, so a caller can share a value with multiple parts of the program without worrying that one of them will mutate it out from under the others, and so nothing about a collection's contents needs a lock or synchronization to read safely from multiple threads.

A mutable collection (`ArrayBuffer`, `mutable.Map`, `mutable.Set`) is still a reasonable tool as a private implementation detail inside a single function — building a large result incrementally with `ArrayBuffer` is often clearer and faster than a purely recursive construction. The rule is about the boundary, not the inside of a method body: a mutable collection built locally must never be returned, stored in a field, or passed to another function directly — convert it to its immutable counterpart (`.toList`, `.toVector`, `.toMap`, or `.result()` on a builder) before it leaves the scope that built it.

## Bad Example
```scala
import scala.collection.mutable

class OrderBook:
  val orders: mutable.Map[String, Order] = mutable.Map.empty // shared mutable state
  def add(order: Order): Unit = orders(order.id) = order
  def snapshot: mutable.Map[String, Order] = orders // caller can mutate our internals
```

## Good Example
```scala
case class OrderBook(orders: Map[String, Order] = Map.empty):
  def add(order: Order): OrderBook = copy(orders = orders + (order.id -> order))

def buildIndex(orders: List[Order]): Map[String, Order] =
  val builder = scala.collection.mutable.Map.empty[String, Order] // local, never escapes
  orders.foreach(o => builder(o.id) = o)
  builder.toMap
```

## Notes
- `List.of`/`Map.of`-equivalents in Scala are the literal constructors (`List(1, 2, 3)`, `Map("a" -> 1)`) — they already produce immutable collections, so there is no separate "immutable factory" to remember to call.
- Passing a mutable collection into a method that expects `Seq`/`Map`/`Set` compiles, because the mutable types are subtypes of the immutable read interfaces — this is a trap, not a safety net; it lets a mutable instance leak through an immutable-looking API without a compile error.
- `Vector` is the general-purpose immutable sequence for both random access and append/prepend; reach for `List` specifically for head/tail recursion and pattern matching, not as a default "any sequence" type.
- `immutability-avoid-mutable-builder-escape` covers the specific failure mode of a mutable accumulator leaking past its construction scope.

## References
- [Scala Collections — Overview](https://docs.scala-lang.org/overviews/collections-2.13/overview.html)
