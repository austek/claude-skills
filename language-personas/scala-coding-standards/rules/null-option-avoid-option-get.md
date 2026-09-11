---
title: Never Call .get on an Option
impact: CRITICAL
impactDescription: eliminates the exact unchecked runtime exception Option exists to prevent
tags: [option, nullability, anti-pattern]
---

# Never Call .get on an Option [CRITICAL]

## Description
`Option.get` returns the wrapped value when the `Option` is `Some`, and throws `NoSuchElementException` when it is `None`. Calling it reintroduces the precise failure mode that using `Option` was supposed to remove: an unchecked runtime exception at the point of use, with nothing in the type system forcing the caller to have handled the empty case first. Code that writes `.get` has, in effect, converted a compiler-enforced check back into a manual one that the compiler can no longer verify was actually done — indistinguishable, from the type checker's perspective, from calling `.get` on an `Option` nobody ever checked.

Every legitimate use of `.get` has a safer equivalent that makes the same intent explicit without the crash risk: `getOrElse(default)` for a fallback value, `fold(onEmpty)(onValue)` to handle both branches with different logic, pattern matching (`case Some(x) => ...; case None => ...`) when the branches differ significantly in shape, or simply staying inside `map`/`flatMap`/a `for`-comprehension so the value never needs to be unwrapped by hand at all. If `.get` is being reached for because "the value is definitely there," that certainty belongs in the type — narrow the `Option` away entirely (e.g. via a guard earlier in the function) rather than asserting it with `.get`.

## Bad Example
```scala
def firstItemName(order: Order): String =
  order.items.headOption.get.name // throws NoSuchElementException on an empty order

val config = loadConfig()
val timeout = config.get("timeout").get.toInt // two silent failure points in one line
```

## Good Example
```scala
def firstItemName(order: Order): Option[String] =
  order.items.headOption.map(_.name)

val timeout = config.get("timeout")
  .flatMap(_.toIntOption)
  .getOrElse(defaultTimeoutSeconds)
```

## Notes
- Test code is the one accepted exception: asserting `result.get` (or `.value` from a test-support extension) against a fixture that is known by construction to be `Some` is a reasonable fail-fast if the fixture itself is ever wrong — production code paths should never carry the same risk.
- Static analysis tools flag this reliably — Wartremover's `OptionPartial` and Scalafix's `DisableSyntax` rule both catch `.get` on `Option` — worth enabling in CI rather than relying on review alone.
- `.getOrElse` is not the same operation as `.get` with a fallback bolted on after the fact — prefer writing `.getOrElse(default)` directly over `if opt.isDefined then opt.get else default`.
- `null-option-orelse-chains-over-nested-match` covers composing several `Option`-returning fallbacks together without ever needing `.get`.

## References
- [Scala 3 Standard Library — Option](https://www.scala-lang.org/api/current/scala/Option.html)
