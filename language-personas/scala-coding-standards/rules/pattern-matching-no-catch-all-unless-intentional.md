---
title: Don't Add a case _ Catch-All Unless the Fallback Is Genuinely Intentional
impact: HIGH
impactDescription: prevents a new variant from being silently routed into the wrong default behavior
tags: [pattern-matching, sealed-trait, exhaustiveness, anti-pattern]
---

# Don't Add a case _ Catch-All Unless the Fallback Is Genuinely Intentional [HIGH]

## Description
Sealing a hierarchy buys exhaustiveness checking (`pattern-matching-sealed-trait-exhaustiveness`) only for as long as no `match` over it ends in a wildcard `case _ => ...`. A trailing `_` compiles happily against any set of remaining variants, present or future, which is exactly the problem: it silences the compiler's "match may not be exhaustive" warning permanently at that call site, so adding a new variant to the sealed hierarchy later no longer breaks the build — it just falls through into whatever the wildcard does, which was written to handle a different, smaller set of cases and is very often wrong for the new one. The wildcard is usually added under time pressure, to make a warning go away without addressing it — and it removes the exact safety net sealing the hierarchy was meant to install.

A wildcard is legitimate in two situations, and both should be visibly deliberate: matching on a genuinely open type where the full set of values isn't knowable at compile time (a `String`, an arbitrary `Int` range, an external enum) — there sealing doesn't apply and exhaustiveness was never on the table; or a documented default over a sealed type where new cases falling into that default is the actual intended behavior, not an oversight — in which case a one-line comment naming that intent belongs next to the `case _`, so a future reader can tell "this was decided" from "this was missed."

## Bad Example
```scala
enum PaymentStatus:
  case Pending, Authorized, Captured, Refunded

def isFinal(status: PaymentStatus): Boolean = status match
  case PaymentStatus.Captured => true
  case PaymentStatus.Refunded => true
  case _                      => false
  // adding PaymentStatus.Disputed later compiles silently and is classified `false`
  // with no signal that Disputed was never actually considered
```

## Good Example
```scala
enum PaymentStatus:
  case Pending, Authorized, Captured, Refunded

def isFinal(status: PaymentStatus): Boolean = status match
  case PaymentStatus.Captured  => true
  case PaymentStatus.Refunded  => true
  case PaymentStatus.Pending   => false
  case PaymentStatus.Authorized => false
  // no default — adding a new PaymentStatus forces this match to be revisited
```

## Notes
- On an open, non-sealed scrutinee — an `Int`, a `String`, a third-party type outside the codebase's control — a `case _` default is the normal and correct way to close the match; the rule targets sealed hierarchies specifically, where exhaustiveness is knowable and worth preserving.
- If a wildcard default over a sealed type really is intentional (e.g. "every status added after v2 defaults to non-final until explicitly reviewed"), say so in a short comment on the `case _` line — that turns a silent gap into a documented decision a reviewer can push back on.
- Grouping several variants behind one `case A | B | C =>` alternative pattern is not the same anti-pattern — it still names each covered case explicitly and still fails to compile if a new variant is added and left out.
- `pattern-matching-sealed-trait-exhaustiveness` is the rule this one directly protects — sealing a hierarchy and then wildcarding every match over it gets none of sealing's benefit.

## References
- [Scala 3 Reference — Exhaustivity Checking](https://docs.scala-lang.org/scala3/reference/enums/enums.html)
