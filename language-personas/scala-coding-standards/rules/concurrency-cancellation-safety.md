---
title: Protect Multi-Step Mutations With uncancelable, Not Ad Hoc Hope
impact: HIGH
impactDescription: prevents a cancelled fiber from leaving a multi-step operation half-applied
tags: [concurrency, cancellation, cats-effect, io]
---

# Protect Multi-Step Mutations With uncancelable, Not Ad Hoc Hope [HIGH]

## Description
Cats-effect's `IO` fibers are cancellable at well-defined points, which is exactly what makes `IO.race` and timeouts work correctly (`concurrency-future-vs-io`) — but that same cancellability is dangerous for an operation with more than one step where the steps must succeed or fail together. A `for`-comprehension chaining a debit and a credit, or a file write followed by an index update, has a window between its steps where cancellation can land: if the fiber running it is cancelled after the debit completes but before the credit runs, the operation is left half-applied, with no exception raised and no code path that ever notices — cancellation is not a thrown error, so an ordinary `try`/`finally`-style construct never sees it.

`IO.uncancelable { poll => ... }` closes that window: the body runs with cancellation masked by default, so a cancellation request arriving mid-body waits until the body finishes rather than interrupting it partway through. Where part of that body is safe (even desirable) to interrupt — an initial validation step that hasn't mutated anything yet — wrap just that part in the `poll` function `uncancelable` provides, which reinstates cancellability for that one sub-effect while the rest of the block stays masked. This is the same concern `effect-systems-resource-safety`'s `Resource` addresses for acquire/release specifically; `uncancelable`/`poll` is the general-purpose tool for any multi-step invariant that needs to be atomic with respect to cancellation, not just a resource's lifecycle.

## Bad Example
```scala
import cats.effect.IO

def transferFunds(from: Account, to: Account, amount: BigDecimal): IO[Unit] =
  for
    _ <- validateFunds(from, amount)
    _ <- debit(from, amount)
    _ <- credit(to, amount) // if this fiber is cancelled here, the debit already happened with no matching credit
  yield ()
```

## Good Example
```scala
import cats.effect.IO

def transferFunds(from: Account, to: Account, amount: BigDecimal): IO[Unit] =
  IO.uncancelable { poll =>
    for
      _ <- poll(validateFunds(from, amount)) // safe to interrupt: nothing has mutated yet
      _ <- debit(from, amount)               // masked: this and the credit below run as one atomic unit
      _ <- credit(to, amount)
    yield ()
  }
```

## Notes
- `poll(fa)` reinstates cancellability for exactly the wrapped `fa`, not for the rest of the `uncancelable` block — everything outside a `poll` call stays masked.
- Masking a long-running operation entirely (never calling `poll` at all) defeats the purpose of a cancellable runtime for that stretch of code — reserve full masking for genuinely short, invariant-critical sections, not an entire request handler.
- `effect-systems-resource-safety`'s `Resource`/`bracketCase` already apply this same masking internally around acquire and release; reach for `uncancelable` directly only when the atomicity requirement isn't naturally expressed as a resource.
- ZIO's `ZIO.uncancelable` and Ox's cancellation-aware forks solve the same problem for their respective runtimes, with comparable masking semantics.

## References
- [Cats Effect — Fiber Cancellation](https://typelevel.org/cats-effect/docs/thread-model)
