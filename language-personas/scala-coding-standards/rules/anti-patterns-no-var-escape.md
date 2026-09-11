---
title: Never Let a Local var Escape the Scope That Declared It
impact: HIGH
impactDescription: keeps a mutable binding's lifetime provably bounded to one call, instead of silently outliving it
tags: [anti-patterns, var, mutability, closures]
---

# Never Let a Local var Escape the Scope That Declared It [HIGH]

## Description
`immutability-val-over-var` allows a narrow exception: a local `var` fully contained within a single function, never read or written outside it, for a tight accumulation loop. That exception depends entirely on containment — the moment a `var` is captured by a closure that outlives the function, returned as part of a value, or exposed as a class field, it stops being a private implementation detail and becomes long-lived, uncontrolled mutable state, with none of the safety the "local and contained" carve-out was granting it. A closure capturing a `var` by reference keeps that binding alive (and mutable) for as long as anything holds the closure, and nothing in the closure's type (`() => Int`) signals that calling it twice can return two different answers, or that two separate holders of the same closure share one mutable counter between them.

This is worse than an ordinary shared `var` field, because it's easy to miss in review — the `var` declaration and the escape (the closure literal, the return statement) can be lines apart, and the type signature of what gets returned gives no hint that mutable state is involved at all. Where a value genuinely needs to survive past its declaring scope and be updated afterward, model that requirement explicitly with a concurrency-safe primitive (`cats.effect.Ref`, `concurrency-avoid-shared-mutable-state`) whose mutation is a describable, typed operation, rather than a captured `var` whose mutability is invisible at every call site that touches it.

## Bad Example
```scala
def counter(): () => Int =
  var count = 0 // declared local, but escapes via the closure returned below
  () => { count += 1; count }

val next = counter()
next() // 1
next() // 2 — mutable state now lives as long as anything holds this closure, invisible from its type
```

## Good Example
```scala
import cats.effect.{IO, Ref}

def counter(): IO[Ref[IO, Int]] = Ref.of[IO, Int](0)

for
  ref <- counter()
  a   <- ref.updateAndGet(_ + 1) // 1
  b   <- ref.updateAndGet(_ + 1) // 2
yield (a, b)
```

## Notes
- `immutability-val-over-var` covers the general `val`-over-`var` default and its narrow contained-loop exception; this rule covers what goes wrong once that exception's containment assumption breaks.
- A `var` captured by a closure is a classic source of surprising aliasing bugs even in single-threaded code — two holders of the same closure share the same counter, which is rarely what either caller expects.
- `concurrency-avoid-shared-mutable-state` covers the same escaped-`var` problem specifically once the escape also crosses into concurrent access — the failure mode there (lost updates) is a further consequence of exactly this rule being violated.
- A `var` returned as a mutable field on an otherwise-immutable-looking object is the same escape by another route — the fix is the same: don't return or capture it, model the mutation explicitly.

## References
- [Scala 3 Book — Immutable Values](https://docs.scala-lang.org/scala3/book/taste-vars-data-types.html)
