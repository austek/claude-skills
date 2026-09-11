---
title: Prefer IO Over Future for New Concurrent Code
impact: HIGH
impactDescription: replaces an eager, unmemoized-by-default, uncancellable computation with one that is lazy, referentially transparent, and cancellable
tags: [concurrency, future, io, cats-effect]
---

# Prefer IO Over Future for New Concurrent Code [HIGH]

## Description
`scala.concurrent.Future` and `cats.effect.IO` both represent an asynchronous computation, but they differ in ways that matter for every piece of code built on top of them. A `Future` starts running the instant it's constructed — `Future { sideEffect() }` has already begun executing on whatever `ExecutionContext` is in implicit scope before the next line of code runs — and that `ExecutionContext` has to be available as a `using`/implicit parameter at every call site that creates one, threading a scheduling decision through code that shouldn't need to care about it (`concurrency-execution-context-explicit`). An `IO[A]` value, by contrast, is a description of a computation: constructing it runs nothing, and the same `IO` value can be passed around, stored, and reused freely, with the described effect only happening once something actually runs it (`effect-systems-io-for-side-effects`).

That laziness compounds with two further differences. A `Future` is cancellable in name only — `Future` offers no built-in way to stop a running computation once started — while `IO`'s fiber-based runtime supports genuine cancellation, propagated through `IO.race`, timeouts, and structured scopes. And a `Future`'s result is memoized the moment it completes (every observer of the same `Future` value sees the one execution's result), whereas re-running the same `IO` value re-executes its description from scratch each time — this is what keeps `IO` referentially transparent (`effect-systems-referential-transparency`); memoizing a specific `IO` value is available via `.memoize` when that behavior is actually wanted, rather than being the unconditional default. For new code with a choice, `IO` (or an equivalent structured effect type) gives strictly more control with no corresponding downside `Future` avoids.

## Bad Example
```scala
import scala.concurrent.{ExecutionContext, Future}

def fetchUser(id: UserId)(using ec: ExecutionContext): Future[User] =
  Future { database.queryUser(id) } // starts running immediately on construction; cannot be cancelled or retried as a value
```

## Good Example
```scala
import cats.effect.IO

def fetchUser(id: UserId): IO[User] =
  IO.blocking(database.queryUser(id)) // a description; runs (and can be cancelled or retried) only once actually executed
```

## Notes
- `Future` is unavoidable at boundaries that require it — some older libraries and parts of the standard library only speak `Future` — and cats-effect provides `IO.fromFuture` to lift one into `IO` at exactly that boundary.
- Cancellation is the sharpest practical difference: an `IO` fiber racing a timeout (`IO.race(op, IO.sleep(5.seconds))`) actually stops the losing computation; the equivalent `Future` keeps running to completion regardless of which one you decided to use.
- `concurrency-execution-context-explicit` covers the specific cost of `Future`'s implicit `ExecutionContext` requirement; `IO`'s runtime is configured once, at the program's edge, rather than threaded through every signature.
- This is a default for new code, not a mandate to rewrite working `Future`-based code with no other reason to touch it.

## References
- [Cats Effect — Getting Started](https://typelevel.org/cats-effect/docs/getting-started)
