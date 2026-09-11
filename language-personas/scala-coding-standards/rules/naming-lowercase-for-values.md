---
title: Use lowerCamelCase for Values, Methods, and Parameters
impact: LOW
impactDescription: lets a reader tell a value from a type on sight, with no need to check where it was declared
tags: [naming, style, conventions]
---

# Use lowerCamelCase for Values, Methods, and Parameters [LOW]

## Description
Scala's casing conventions carry real information: a lowercase-initial identifier (`maxRetries`, `processOrder`, `userId`) is a value, method, or parameter — something you call or read — while an uppercase-initial identifier is a type. This isn't just cosmetic; the parser and a reader both lean on it. Pattern matching treats an uppercase-initial identifier in a pattern as a stable reference to bind against (a case object, a constant) rather than as a fresh binding, so a `var`/`val` named against convention (`val UserId = "u-1"`) can produce a pattern match that silently compares against the wrong thing instead of binding a new name — one more reason the convention isn't optional style.

Apply `lowerCamelCase` uniformly to `val`s, `var`s, `def`s, and parameters: `maxRetries`, not `MaxRetries` or `max_retries`; `processOrder`, not `ProcessOrder`. The one commonly accepted exception is a genuine compile-time constant declared inside an `object` (`final val MaxRetries = 3`), where the Scala style guide permits `UpperCamelCase` to signal "this is a fixed constant, not an ordinary mutable-feeling binding" — that convention is about signaling constant-ness specifically, not a general license to capitalize values, and an ordinary `val` holding a runtime-computed or mutable-feeling value should stay lowercase regardless of how it's used.

## Bad Example
```scala
val MaxRetries = 3 // reads like a type or a stable pattern-match reference at the call site
def ProcessOrder(order: Order): Receipt = ???
```

## Good Example
```scala
val maxRetries = 3
def processOrder(order: Order): Receipt = ???
```

## Notes
- An uppercase-initial name in a `match` pattern binds against that name's existing value (a stable identifier), not a fresh variable — this is exactly why value/parameter names must stay lowercase-initial to behave the way a reader expects in a pattern.
- `naming-uppercase-for-types` covers the mirror-image convention for classes, traits, and type parameters.
- The single accepted exception is a `final val` constant inside an `object`, by long-standing convention — it does not extend to ordinary `val`s or `var`s.
- Acronyms inside a `lowerCamelCase` name follow the same casing as any other word boundary: `parseJson`, not `parseJSON`; `httpClient`, not `hTTPClient`.

## References
- [Scala Style Guide — Naming Conventions](https://docs.scala-lang.org/style/naming-conventions.html)
