---
title: Prefer val Over var
impact: CRITICAL
impactDescription: eliminates an entire class of reassignment bugs and makes local reasoning sound
tags: [immutability, val, var, functional]
---

# Prefer val Over var [CRITICAL]

## Description
`val` binds a name to a value once; `var` allows reassignment. Default to `val` everywhere — method bodies, class parameters, local bindings — and treat `var` as an exception that needs a specific justification, not a habit carried over from imperative languages. A `var` breaks equational reasoning: a binding that can change means every read of it must be understood in the context of everything that ran before it, not just the value's definition. It also reopens the concurrency hazards immutability is meant to close — a shared `var` needs synchronization, a `val` never does — and it silently defeats structural equality and safe sharing on any class that exposes one as a field.

Most code that reaches for `var` is really building up a result across steps: a running total, a filtered list, an accumulated string. Scala's collection combinators (`map`, `filter`, `fold`, `foldLeft`) and recursion express exactly that shape without a mutable binding — the accumulation becomes the return value of an expression instead of the side effect of a loop. Where a mutable accumulator is genuinely the clearest or fastest way to build something up, keep it local to the function, never exposed as a `var` field or return value, and prefer a mutable collection (see `immutability-immutable-collections-default`) over a raw `var` of a scalar even there.

## Bad Example
```scala
def sumPositive(numbers: List[Int]): Int =
  var total = 0
  for n <- numbers if n > 0 do
    total += n
  total

class RequestCounter:
  var count: Int = 0
  def increment(): Unit = count += 1
```

## Good Example
```scala
def sumPositive(numbers: List[Int]): Int =
  numbers.filter(_ > 0).sum

case class RequestCounter(count: Int = 0):
  def increment: RequestCounter = copy(count = count + 1)
```

## Notes
- Class parameters and fields should be `val` by default; a `var` field makes every instance mutable shared state the moment it is exposed.
- A tight, profiled hot loop with a provable single-threaded lifetime is the rare legitimate case for a local `var` (e.g. manual array indexing) — justify it with a comment, keep it fully private to the function.
- The Scala compiler does not forbid `var`, so linting (Scalafix, Wartremover's `Var` rule) is the practical enforcement mechanism in a team codebase.
- `immutability-copy-method-for-updates` covers the idiomatic way to produce an "updated" value from a `val`-only case class.

## References
- [Scala 3 Book — Immutable Values](https://docs.scala-lang.org/scala3/book/taste-vars-data-types.html)
