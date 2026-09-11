---
title: A Total-Looking Function Signature Must Not Throw
impact: HIGH
impactDescription: keeps a plain A => B signature trustworthy as actually total, with no hidden partiality a caller has to discover
tags: [anti-patterns, throw, purity, referential-transparency]
---

# A Total-Looking Function Signature Must Not Throw [HIGH]

## Description
`error-handling-no-throw-for-control-flow` targets throwing for *expected* domain outcomes specifically — "user not found" modeled as an exception instead of a `Left`. This rule targets a narrower but equally damaging variant: a function whose signature is an ordinary, pure-looking `A => B` — no `Either`, no `Try`, no effect type anywhere in sight — that can nonetheless throw for some input. That signature makes an implicit promise of totality: every `A` produces a `B`. A `Tier => Double` that throws `IllegalStateException` for one particular `Tier` value breaks that promise invisibly — nothing in the type says the function is partial, so every caller has to either already know (from reading the implementation, or from a production incident) which inputs are unsafe, or wrap every call in a `try`/`catch` defensively, on a function whose signature gave no indication that was ever necessary.

This also breaks referential transparency for reasons `effect-systems-referential-transparency` already covers for effect-returning functions: a function that sometimes returns a `B` and sometimes unwinds the stack cannot be freely substituted by "its result," because for some inputs it has no result to substitute — only a side effect (the throw) to observe. If a case genuinely has no meaningful answer, say so in the type: return `Either[E, B]` and let the caller branch on it, the same way `error-handling-either-for-domain-errors` prescribes for any domain operation with more than one outcome. Reserve an actual `throw` for what it's meant for — a bug, a violated invariant — never for a reachable branch of an otherwise pure computation's own input space.

## Bad Example
```scala
def discountRate(tier: Tier): Double =
  tier match
    case Tier.Gold   => 0.2
    case Tier.Silver => 0.1
    case Tier.Basic  => 0.0
    case Tier.Banned => throw new IllegalStateException("banned customers get no rate") // Tier => Double promised totality; this isn't total
```

## Good Example
```scala
enum DiscountError:
  case CustomerBanned

def discountRate(tier: Tier): Either[DiscountError, Double] =
  tier match
    case Tier.Gold   => Right(0.2)
    case Tier.Silver => Right(0.1)
    case Tier.Basic  => Right(0.0)
    case Tier.Banned => Left(DiscountError.CustomerBanned)
```

## Notes
- `error-handling-no-throw-for-control-flow` covers throwing for *expected* outcomes generally; this rule covers the narrower signal that gives the anti-pattern away — a plain, effect-free return type that implies totality it doesn't actually have.
- `effect-systems-referential-transparency`'s notes already flag this same substitution failure for an `IO`-returning function that also throws directly; this rule is the same concern one level down, for functions that never had an effect type to begin with.
- `pattern-matching-sealed-trait-exhaustiveness` is what should catch a missing case at compile time in the first place — a `throw` inside the last arm of an otherwise-exhaustive match is frequently a sign the match was made exhaustive by adding a throwing catch-all rather than by handling every case with a real value.
- A genuine bug guard (an assertion on an invariant the caller's own code should have already upheld) is still a legitimate `throw`, per `error-handling-no-throw-for-control-flow`'s own carve-out — the distinction is whether the input reaching the throwing branch is a reachable, expected case or a broken precondition.

## References
- [Scala 3 Book — Control Structures](https://docs.scala-lang.org/scala3/book/control-structures.html)
