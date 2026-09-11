---
title: Keep IO at the Edges; Keep Core Logic Pure
impact: MEDIUM
impactDescription: keeps validation, calculation, and business rules testable as plain functions with no effect runtime needed
tags: [effect-systems, architecture, io]
---

# Keep IO at the Edges; Keep Core Logic Pure [MEDIUM]

## Description
An `IO`-returning function can call plain, pure functions freely, but the reverse discipline is what actually keeps a codebase easy to test and reason about: push `IO` (or any effect type) out to the boundary of the program — HTTP handlers, database adapters, a `main`/`IOApp` entry point — and keep the core logic in between as plain functions taking plain values and returning plain values, or `Either[E, A]` for expected domain failures (`error-handling-either-for-domain-errors`). A pricing calculation, a validation rule, or a state transition does not need to know an effect system exists; wrapping it in `IO` anyway just to be consistent with the code around it adds a dependency on an effect runtime to something that should be testable with a plain assertion on its return value, with no `unsafeRunSync()` or test-specific effect runner involved at all.

This "functional core, imperative shell" shape also makes the effectful boundary easier to reason about on its own terms: once the pure core is factored out, the remaining `IO`-wrapped code is mostly sequencing — fetch this, validate it with the pure function, persist the result — rather than business rules and side effects interleaved throughout, which is much harder to unit test without standing up whatever the side effects touch (a database, a filesystem, a clock). The boundary doesn't have to be the outermost layer of the entire application — a single method can have this shape internally, calling out to `IO` only for the specific lines that actually touch the outside world and running everything else as ordinary function calls in between.

## Bad Example
```scala
import cats.effect.IO

def processOrder(cart: Cart): IO[Receipt] =
  for
    _     <- IO.raiseWhen(cart.items.isEmpty)(new IllegalStateException("empty cart")) // domain rule buried inside IO
    total =  cart.items.foldLeft(BigDecimal(0))(_ + _.price) // pure, but only reachable via IO here
    _     <- IO.blocking(paymentGateway.charge(total))
  yield Receipt(cart, total)
```

## Good Example
```scala
import cats.effect.IO

enum CartError:
  case Empty

def priceCart(cart: Cart): Either[CartError, BigDecimal] = // pure core: plain, unit-testable
  Either.cond(
    cart.items.nonEmpty,
    cart.items.foldLeft(BigDecimal(0))(_ + _.price),
    CartError.Empty
  )

def processOrder(cart: Cart): IO[Receipt] = // imperative shell: sequences the pure core with effects
  for
    total <- IO.fromEither(priceCart(cart).left.map(err => new IllegalStateException(err.toString)))
    _     <- IO.blocking(paymentGateway.charge(total))
  yield Receipt(cart, total)
```

## Notes
- The pure core (`priceCart`) is testable with a plain assertion on its return value — no `IO` runtime, no `unsafeRunSync`, no test-specific effect runner needed.
- `IO.fromEither` is the standard way to lift a pure `Either[Throwable, A]` result into `IO`'s error channel once the pure core has produced its answer.
- This mirrors "functional core, imperative shell": the shell's job is sequencing effects and translating between the pure core's types and the effect system's; the core's job is everything else.
- `error-handling-either-for-domain-errors` covers shaping the pure core's own error type; this rule covers where the effect boundary sits relative to that core.

## References
- [Cats Effect — Getting Started](https://typelevel.org/cats-effect/docs/getting-started)
