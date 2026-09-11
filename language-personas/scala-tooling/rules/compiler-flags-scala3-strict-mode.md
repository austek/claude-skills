---
title: Assemble a Strict Scala 3 Flag Profile Beyond the Defaults
impact: HIGH
impactDescription: catches a mistaken cross-type equality comparison and a discarded effect value at compile time, both of which type-check silently under default settings
tags: [compiler-flags, scala3, strictEquality, value-discard]
---

# Assemble a Strict Scala 3 Flag Profile Beyond the Defaults [HIGH]

## Description
Scala 3 has no single "strict mode" flag — a strict baseline is assembled from several independent flags, each closing a different gap the defaults leave open. `-language:strictEquality` opts into multiversal equality: without it, `==` between any two types compiles and falls back to `Any`-based equality that's always `false` for genuinely unrelated types, silently; with it, comparing two types requires a `CanEqual[A, B]` instance in scope (case classes get one via `derives CanEqual` for same-type comparisons automatically), so a comparison between mismatched types becomes a compile error instead of a always-false runtime surprise. `-Wvalue-discard` flags an expression whose non-`Unit` result is discarded — catching, for example, an `IO`-returning call made as a bare statement, whose effect never actually runs because nothing sequences it.

Combined with `-deprecation -feature -Wunused:all -Xfatal-warnings` (covered individually elsewhere in this skill), these form a reasonable strict profile: multiversal equality, no silently-discarded effectful results, no unused code, no deprecated API usage, all fatal.

## Bad Example
```scala
final case class UserId(value: String) derives CanEqual

def isStale(id: UserId, cachedKey: String): Boolean =
  id == cachedKey // compiles without strictEquality; always false, nobody notices
```

## Good Example
```scala
// build.sbt
scalacOptions ++= Seq(
  "-language:strictEquality",
  "-Wvalue-discard",
  "-Wunused:all",
  "-deprecation",
  "-feature",
  "-Xfatal-warnings"
)
```
```scala
final case class UserId(value: String) derives CanEqual

def isStale(id: UserId, cachedId: UserId): Boolean =
  id == cachedId // same type on both sides — compiles under strictEquality too
```

## Notes
- `derives CanEqual` on a case class grants `CanEqual[T, T]` only; a deliberate cross-type comparison needs its own `given CanEqual[A, B] = CanEqual.derived`, which makes the intent explicit at the definition site instead of implicit at every call site.
- Adopting `strictEquality` on an existing codebase surfaces every accidental cross-type comparison at once — pair with the same incremental-rollout discipline as [`compiler-flags-xfatal-warnings`](compiler-flags-xfatal-warnings.md) if the initial violation count is large.
- `-Wvalue-discard` is especially relevant alongside effect types — see [`effect-systems-io-for-side-effects`](../../scala-coding-standards/rules/effect-systems-io-for-side-effects.md) in scala-coding-standards for the pattern it's guarding against.

## References
- [Scala 3 Reference — Multiversal Equality](https://docs.scala-lang.org/scala3/reference/contextual/multiversal-equality.html)
