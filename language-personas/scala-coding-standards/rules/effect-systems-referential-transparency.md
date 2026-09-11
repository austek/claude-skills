---
title: Keep Effect-Returning Functions Referentially Transparent
impact: HIGH
impactDescription: makes an IO-returning expression safe to bind to a val, pass around, or reorder without changing behavior
tags: [effect-systems, referential-transparency, io]
---

# Keep Effect-Returning Functions Referentially Transparent [HIGH]

## Description
An expression is referentially transparent when it can be replaced by its value everywhere it appears without changing the program's behavior. A pure function call like `add(2, 3)` has this trivially — replacing it with `5` changes nothing. `IO[A]` is designed to preserve the same property for effectful code: `IO.delay(println("hi"))` is a *description* of an effect, and that description is itself a pure value — evaluating it, binding it to a `val`, passing it to a function, storing it in a data structure, never runs the `println`. Only an explicit run — `.unsafeRunSync()`, or the runtime driving an `IOApp` — causes the described side effect to actually happen. An `IO[A]` value can therefore be bound to a `val`, passed around, and reused freely, the same way any other pure value can, without accidentally triggering or duplicating the underlying side effect.

This guarantee breaks the moment a "pure-looking" function performs a side effect directly instead of returning an `IO` describing one — printing inside an otherwise-`A`-returning method, mutating a `var` as a side channel, throwing instead of surfacing the failure through `Either`/an `IO`'s own error channel. Any of these makes the function's result depend on *when* and *how many times* it's called, not just its arguments, which defeats substitution: calling it twice is not the same as calling it once and reusing the value, and reordering two such calls can change the program's observable behavior. Keep every side effect behind an `IO`-returning boundary (`effect-systems-io-for-side-effects`), and keep everything else — validation, calculation, data transformation — as plain functions the compiler can already treat as pure.

## Bad Example
```scala
var callCount = 0

def nextRequestId(): String =
  callCount += 1 // side channel: the result depends on how many times this has already run
  s"req-$callCount"
```

## Good Example
```scala
import cats.effect.{IO, Ref}

def nextRequestId(counter: Ref[IO, Int]): IO[String] =
  counter.modify(n => (n + 1, s"req-${n + 1}")) // the mutation is itself a described, sequenced effect
```

## Notes
- `IO[A]` values are themselves pure — constructing one, no matter how many times, never runs the described effect; only an explicit run does.
- `cats.effect.Ref[F, A]` is the effect-system-safe way to model mutable state several concurrent fibers might share — the mutation becomes a describable, sequenced `F[_]` operation instead of a raw `var`.
- A function returning `IO[A]` that also throws directly, rather than surfacing the error inside the `IO`, breaks the same substitution guarantee for its error path — see `error-handling-io-for-effectful-errors`.
- `effect-systems-io-for-side-effects` covers the mechanics of wrapping a side effect in `IO`; this rule covers why doing so consistently, with nothing sneaking out around it, is what the runtime's other guarantees (resource safety, correct concurrency) actually depend on.

## References
- [Cats Effect — Ref](https://typelevel.org/cats-effect/docs/std/ref)
