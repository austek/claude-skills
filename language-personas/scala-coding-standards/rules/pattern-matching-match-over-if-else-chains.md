---
title: Prefer match Over Long if/else if Chains
impact: MEDIUM
impactDescription: replaces a right-leaning conditional pyramid with a flat, exhaustiveness-checkable list
tags: [pattern-matching, readability, control-flow]
---

# Prefer match Over Long if/else if Chains [MEDIUM]

## Description
When branching on the value or shape of a single expression across three or more mutually exclusive conditions, use `match` rather than an `if`/`else if` chain. `match` names the thing being branched on exactly once, at the top, and lists every branch as a flat, parallel `case`, instead of accumulating one more `else if` per condition into a chain that leans further right with each addition and gets harder to scan the longer it grows. Beyond readability, `match` composes with the rest of Scala's pattern language in ways an `if` chain cannot: patterns can destructure the scrutinee's shape (`pattern-matching-destructuring-over-accessors`) and attach guards (`pattern-matching-guard-clauses`) in the same construct, and — critically — when the scrutinee is a sealed type or `enum`, the compiler checks the `match` for exhaustiveness (`pattern-matching-sealed-trait-exhaustiveness`), something an `if`/`else if` chain never gets regardless of how carefully it is written.

`if`/`else` remains the right tool for a single binary condition, or for a handful of branches that test genuinely unrelated conditions rather than different values or shapes of one common scrutinee — `match` has nothing to offer when there is no single value being discriminated on. But once there is such a value, prefer `match` even for as few as two cases if the scrutinee is a sealed type and more variants are plausible later: the exhaustiveness check that comes with it has no equivalent in `if`/`else`, and it is exactly the protection that keeps a future added variant from being missed silently.

## Bad Example
```scala
def httpStatusCategory(code: Int): String =
  if code >= 200 && code < 300 then "success"
  else if code >= 300 && code < 400 then "redirect"
  else if code >= 400 && code < 500 then "client-error"
  else if code >= 500 && code < 600 then "server-error"
  else "unknown"
```

## Good Example
```scala
def httpStatusCategory(code: Int): String = code match
  case c if c >= 200 && c < 300 => "success"
  case c if c >= 300 && c < 400 => "redirect"
  case c if c >= 400 && c < 500 => "client-error"
  case c if c >= 500 && c < 600 => "server-error"
  case _                        => "unknown"
```

## Notes
- On a sealed type or `enum`, prefer `match` over `if`/`else` even for two branches — the exhaustiveness check it buys has no equivalent in an `if` chain, and the benefit only grows as variants are added later.
- `if`/`else` is still the clearer choice for one condition with two outcomes, or for branches driven by unrelated conditions rather than one shared scrutinee — reaching for `match` there adds ceremony without adding safety.
- `match` expressions are, well, expressions — they return a value directly and typically replace both the branching and a separate "assign the result" step that an `if`/`else if` chain often needs a `var` for (see `immutability-val-over-var`).
- `pattern-matching-guard-clauses` covers writing the individual conditions inside each `case` cleanly once the chain has been converted to a `match`.

## References
- [Scala 3 Book — Match Expressions](https://docs.scala-lang.org/scala3/book/control-structures.html#match-expressions)
