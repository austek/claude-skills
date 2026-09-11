---
title: Use Guard Clauses Inside match Instead of Nested Conditionals
impact: MEDIUM
impactDescription: keeps each case's full condition — shape and predicate — visible in one line
tags: [pattern-matching, guards, readability]
---

# Use Guard Clauses Inside match Instead of Nested Conditionals [MEDIUM]

## Description
When a `case` needs to test something beyond the shape of the pattern — a bound value being positive, a string matching a format, a collection being non-empty — attach that test directly to the case with an `if` guard (`case Some(n) if n > 0 => ...`) rather than matching broadly on the shape alone and nesting a separate `if`/`else` inside the case body. A guard keeps a case's complete condition — both "does this match the shape" and "does the extra predicate hold" — visible in the single line that introduces the case, so scanning the list of `case` lines shows every branch's full condition at a glance. Pushing the predicate into the body instead splits that condition across two places and, once there is more than one extra check, tends to grow into an `if`/`else` pyramid nested one level inside an otherwise flat `match`.

A guard is evaluated only after its case's pattern has already matched, so it can freely reference names the pattern just bound (`n` in `case Some(n) if n > 0`). This also creates a common exhaustiveness trap worth watching for deliberately: a guarded case does not cover the situations where the shape matches but the guard is false, so `case Some(n) if n > 0 => positive(n)` alone is not exhaustive over `Option[Int]` even though it looks, at a glance, like it handles every `Some`. Always pair a guarded case with either its complement (`case Some(n) => nonPositive(n)`) or a case that legitimately doesn't need the guard.

## Bad Example
```scala
def classify(value: Option[Int]): String =
  value match
    case Some(n) =>
      if n > 0 then "positive"
      else if n < 0 then "negative"
      else "zero"
    case None => "absent"
```

## Good Example
```scala
def classify(value: Option[Int]): String =
  value match
    case Some(n) if n > 0 => "positive"
    case Some(n) if n < 0 => "negative"
    case Some(_)          => "zero"
    case None             => "absent"
```

## Notes
- A guard is checked only once the pattern itself has matched — if the guard is false, matching continues to the next `case` rather than failing the whole `match`, which is exactly what makes chained guarded cases like the example above work.
- Every guarded case needs its complement covered somewhere in the same `match` (a fallback guard, an un-guarded case for the same shape, or an explicit case for the negated condition) — the compiler's exhaustiveness check cannot see into an arbitrary boolean guard, so it will not warn about a gap a guard leaves open.
- Prefer a guard over splitting one logical case into two different, differently-shaped patterns purely to encode a condition the original shape already captures well enough with an `if`.
- `pattern-matching-destructuring-over-accessors` covers the companion technique of binding multiple fields in the pattern itself, which guards then test against directly.

## References
- [Scala 3 Reference — Pattern Matching](https://docs.scala-lang.org/scala3/reference/changed-features/pattern-matching.html)
