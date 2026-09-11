---
title: Use Ox's Direct-Style supervised/fork for Structured Concurrency
impact: MEDIUM
impactDescription: expresses concurrent forks as plain sequential-looking code, with no monadic effect wrapper needed
tags: [effect-systems, ox, direct-style, structured-concurrency]
---

# Use Ox's Direct-Style supervised/fork for Structured Concurrency [MEDIUM]

## Description
Cats-effect's `IO` and ZIO both use a monadic encoding — an effect is a value, sequenced with `flatMap`/a `for`-comprehension, and only actually runs when an outer runtime executes it. Ox takes a different approach, made possible by JDK virtual threads (Project Loom): concurrent code is written in *direct style* — ordinary method calls that block the calling virtual thread rather than an effect value that gets sequenced — while still getting the safety guarantees a structured effect system provides, chiefly that a child computation can never outlive the scope that started it. `supervised { ... }` opens such a scope; `fork { ... }` inside it starts a concurrent computation and returns a handle whose `.join()` blocks — cheaply, since it blocks a virtual thread rather than an OS thread — until that fork completes. When the `supervised` block exits, every fork started inside it has been joined or cancelled; there is no way for a forked computation to leak past the block that created it, which is the "structured" half of structured concurrency.

This trades away referential transparency in exchange for code that reads like ordinary sequential Scala, with no `.map`/`.flatMap` chains and no separate effect type to learn. What Ox keeps from the monadic effect systems is the *structure*: a fork failing inside a `supervised` block cancels its sibling forks and propagates the failure to the caller — the same safety cats-effect's structured concurrency operators or ZIO's fibers provide, without wrapping every intermediate value in an effect type to get it.

## Bad Example
```scala
// unstructured: the thread can outlive its caller, and its result has no way to be retrieved
def fetchBoth(): (Int, Int) =
  val t = new Thread(() => computeB())
  t.start()
  val a = computeA() // if this throws, the thread keeps running with no one watching it
  t.join()
  (a, 0) // computeB's actual result was discarded — there was never a way to retrieve it
```

## Good Example
```scala
import ox.{supervised, fork}

def fetchBoth(): (Int, Int) =
  supervised {
    val forkB = fork { computeB() }
    val a = computeA() // a failure here cancels forkB before the scope exits
    val b = forkB.join()
    (a, b)
  }
```

## Notes
- `supervised` guarantees every fork it starts is joined or cancelled before the block returns — a forked computation can never outlive its enclosing scope.
- Ox requires a JDK with virtual thread support (JDK 21+); `fork`'s cheap blocking `.join()` depends on it.
- This is a genuinely different model from cats-effect `IO`/ZIO, not a drop-in replacement — pick one effect model per codebase rather than mixing direct-style Ox forks with monadic `IO` chains in the same call path.
- `effect-systems-resource-safety` and `effect-systems-no-blocking-inside-effect` are framed around monadic effect systems specifically; Ox's structured scopes solve the resource-lifetime half of that problem differently, through the scope itself.

## References
- [Ox — Documentation](https://ox.softwaremill.com)
