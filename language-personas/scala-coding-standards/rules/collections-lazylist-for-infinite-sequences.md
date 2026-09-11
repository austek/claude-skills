---
title: Use LazyList for Self-Referential or Infinite Sequences
impact: LOW
impactDescription: expresses an unbounded or self-referential sequence without pre-computing more than what's consumed
tags: [collections, lazylist, laziness]
---

# Use LazyList for Self-Referential or Infinite Sequences [LOW]

## Description
Some sequences are naturally defined in terms of themselves — the Fibonacci sequence, an unbounded stream of sensor readings, a retry sequence with exponentially growing delays — where writing them as a strict `List` is either impossible (there is no finite end to compute) or wasteful (only the first handful of elements will ever be consumed). `LazyList` is Scala's collection for exactly this: like `List` it has a head and a tail, but the tail is computed on demand, the first time something asks for it, rather than eagerly at construction — which is what makes a self-referential definition like the Fibonacci sequence expressible as a value at all: only as many terms as actually get consumed are ever computed.

The detail that most distinguishes `LazyList` from a plain `Iterator` (also lazy) is memoization: once a `LazyList` cell's value is forced, that value is cached on the `LazyList` itself, so re-traversing it — or holding a reference to an earlier point and consuming forward from there again — recomputes nothing. This is convenient for reuse but is also the sharpest way to misuse a `LazyList`: keeping a reference to the *head* of a long or infinite `LazyList` while consuming far into its tail forces that whole traversed prefix to stay cached in memory for as long as the head reference is reachable, defeating the memory benefit laziness was supposed to provide. When a sequence will only ever be walked once, forward, with nothing kept around afterward, a plain `Iterator` gives the same on-demand evaluation without paying for memoization at all.

## Bad Example
```scala
val naturals: LazyList[Int] = LazyList.from(1)

def sumFirstEvenSquaresOver(bound: Int): BigInt =
  naturals // a one-shot scan, but naturals is a shared val — every forced cell stays memoized on it
    .map(n => BigInt(n) * n)
    .filter(_ % 2 == 0)
    .takeWhile(_ <= bound)
    .sum
```

## Good Example
```scala
def sumFirstEvenSquaresOver(bound: Int): BigInt =
  Iterator.from(1) // one-shot, forward-only — nothing is memoized after this call returns
    .map(n => BigInt(n) * n)
    .filter(_ % 2 == 0)
    .takeWhile(_ <= bound)
    .sum

val fibs: LazyList[BigInt] =
  BigInt(0) #:: BigInt(1) #:: fibs.zip(fibs.tail).map((a, b) => a + b) // memoization is the point here
```

## Notes
- `LazyList` memoizes every forced element on the list itself; holding a reference to an early cell while consuming far ahead keeps that whole prefix in memory.
- `Iterator` is lazy the same way but never memoizes — prefer it for a single forward, one-shot traversal with nothing kept afterward.
- `#::` builds a `LazyList` cell without forcing its tail, which is what makes a self-referential definition like `fibs` possible — the recursive reference is only evaluated once something asks for that element.
- `collections-view-for-lazy-chains` covers the related but distinct tool for deferring a transformation chain over an already-finite, already-materialized collection.

## References
- [Scala Standard Library — LazyList](https://www.scala-lang.org/api/current/scala/collection/immutable/LazyList.html)
