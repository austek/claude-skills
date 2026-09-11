---
title: Wrap Side Effects in IO Instead of Running Them Directly
impact: CRITICAL
impactDescription: makes "this can affect the outside world" part of the type, instead of something only found by reading the body
tags: [effect-systems, cats-effect, io]
---

# Wrap Side Effects in IO Instead of Running Them Directly [CRITICAL]

## Description
A method's return type answers "what does this produce," but nothing about an ordinary `Int` or `Unit` return type says whether producing it also wrote to a file, made a network call, or printed to a console. Cats-effect's `IO[A]` closes that gap: `IO[A]` is a description of a computation that, when run, produces an `A` and may perform side effects along the way — constructing an `IO` value does nothing by itself; only an explicit run (`unsafeRunSync()` at a program's entry point, or the runtime managing an `IOApp`) actually executes the described effect. Building side-effecting logic as `IO` values instead of plain methods that perform the effect as a statement means the type signature itself documents which operations can affect the outside world (`IO[A]`) versus which are pure computations (a plain `A`), and composing several effectful steps with `.map`/`.flatMap`/a `for`-comprehension (`for-comprehensions-monadic-composition`) sequences them without any of them running early.

`IO.pure(a)` lifts an already-computed, effect-free value into `IO` when a signature requires one; `IO.delay(sideEffectingExpr)` (also spelled `IO(sideEffectingExpr)`) wraps an expression whose *evaluation itself* has a side effect, deferring that evaluation until the `IO` runs rather than performing it while the `IO` value is being constructed. Reach for one of those two constructors at the boundary where a side effect is unavoidable — a database call, a log line, reading a clock — rather than performing the effect directly inside an otherwise pure-looking function and returning an already-produced value.

## Bad Example
```scala
def logAndSave(order: Order): Unit =
  println(s"saving order ${order.id}") // runs immediately, as soon as logAndSave is called
  database.save(order) // also runs immediately — no way to sequence, retry, or defer this
```

## Good Example
```scala
import cats.effect.IO

def logAndSave(order: Order): IO[Unit] =
  for
    _ <- IO.println(s"saving order ${order.id}") // describes the effect; nothing runs yet
    _ <- IO.delay(database.save(order))           // same — deferred until this IO is run
  yield ()
```

## Notes
- Constructing an `IO` value performs no side effect by itself; only running it does — this is what makes `IO` values referentially transparent (`effect-systems-referential-transparency`) despite describing effectful work.
- `IO.println` is a built-in convenience equivalent to `IO.delay(println(...))`.
- Keep this boundary at the edges of the program (`effect-systems-effect-boundary-at-edges`) — a function's core logic can stay plain and pure, with `IO` wrapping only the parts that truly touch the outside world.
- ZIO's `ZIO[R, E, A]` (and its `Task[A]` alias) follows the same construction discipline (`ZIO.succeed`, `ZIO.attempt`) for the same reason.

## References
- [Cats Effect — Getting Started](https://typelevel.org/cats-effect/docs/getting-started)
