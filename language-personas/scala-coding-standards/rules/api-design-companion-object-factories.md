---
title: Construct Through Companion Object Factories, Not a Public Constructor
impact: MEDIUM
impactDescription: gives every named construction path (defaults, common variants) a self-documenting call site instead of a positional argument list
tags: [api-design, companion-object, factories, construction]
---

# Construct Through Companion Object Factories, Not a Public Constructor [MEDIUM]

## Description
Scala lets a class's companion `object` define an `apply` method that stands in for `new`, so `RetryPolicy(3, 500.millis)` and `new RetryPolicy(3, 500.millis)` can both compile — but only the first reads as ordinary function application, with `new` never appearing at any call site. That matters beyond style: once construction goes through the companion, the companion can expose more than one named entry point for the common ways callers actually build the type — `RetryPolicy.none`, `RetryPolicy.default`, `Config.fromEnv` — each naming what it produces, instead of every variant collapsing into the same positional constructor call with a caller left to infer meaning from which arguments they passed and which they defaulted.

This also decouples the public construction API from the type's internal field layout: the primary constructor can stay `private`, its parameter list can be reordered or extended, and none of that touches the named factories' signatures or the call sites that use them. A single `apply` alone doesn't buy much over a public constructor when there's only one way to build the type — the real payoff shows up once a type has more than one legitimate construction path, and the companion gives each one a name instead of forcing every variant through the same argument list with different defaults filled in by hand.

## Bad Example
```scala
import scala.concurrent.duration.*

class RetryPolicy(val maxAttempts: Int, val backoff: FiniteDuration, val jitter: Boolean)

new RetryPolicy(3, 500.millis, true) // "new" is exposed, and a "no retry" variant means spelling out every field again
new RetryPolicy(1, Duration.Zero, false)
```

## Good Example
```scala
import scala.concurrent.duration.*

final class RetryPolicy private (val maxAttempts: Int, val backoff: FiniteDuration, val jitter: Boolean)

object RetryPolicy:
  def apply(maxAttempts: Int, backoff: FiniteDuration): RetryPolicy =
    new RetryPolicy(maxAttempts, backoff, jitter = true)

  val none: RetryPolicy =
    new RetryPolicy(maxAttempts = 1, backoff = Duration.Zero, jitter = false)

RetryPolicy(3, 500.millis)
RetryPolicy.none
```

## Notes
- The primary constructor can be `private` (as here) or left public — making it private is what actually forces every caller, inside and outside the file, through the named factories, closing off the positional-argument path entirely.
- `api-design-smart-constructors` covers the case where a factory also validates and returns `Either[E, T]`; a plain factory like `RetryPolicy.apply` above is for construction with no failure mode, just named defaults.
- Named `val`s (`RetryPolicy.none`) work well for fixed, argument-free variants; reach for a `def` factory as soon as the variant needs even one parameter.
- Overloading `apply` itself (several `apply` signatures) is fine for a handful of genuinely equivalent shapes, but once the variants mean different things (not just different argument types for the same meaning), distinct names read better than overload resolution a caller has to reverse-engineer.

## References
- [Scala 3 Book — Companion Objects](https://docs.scala-lang.org/scala3/book/domain-modeling-tools.html#companion-objects)
