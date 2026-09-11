---
title: Destructure with Pattern Matching Instead of Chained Accessors
impact: MEDIUM
impactDescription: pulls every needed field from one match instead of repeating a navigation path per field
tags: [pattern-matching, destructuring, readability, case-class]
---

# Destructure with Pattern Matching Instead of Chained Accessors [MEDIUM]

## Description
A case class's compiler-generated `unapply` makes it directly usable in a pattern, so pulling several fields — including fields nested inside other case classes — out of a value is a single destructuring step rather than a sequence of accessor calls. `case Event(Payload(Header(timestamp, source), body), _) => ...` binds `timestamp`, `source`, and `body` in one shot, naming exactly the shape being relied on at the point of use; the equivalent written as `event.payload.header.timestamp`, `event.payload.header.source`, `event.payload.body` repeats the same navigation path once per field, and gives no single place that documents "this function depends on `Header` having a `timestamp` and a `source`."

Destructuring also scopes cleanly: the bound names (`timestamp`, `source`) exist only where the pattern introduced them, short local names standing in for what would otherwise be a longer accessor chain repeated at every use. This matters most when a function genuinely needs several fields from the same nested value together — for a single top-level field, a plain accessor (`event.id`) is still the simplest, most direct choice, and reaching for a full pattern match to extract one field is unnecessary ceremony.

## Bad Example
```scala
def summarize(event: Event): String =
  val ts = event.payload.header.timestamp
  val src = event.payload.header.source
  val bodySize = event.payload.body.length
  s"[$ts] from $src (${bodySize} bytes)"
```

## Good Example
```scala
def summarize(event: Event): String =
  event match
    case Event(Payload(Header(timestamp, source), body), _) =>
      s"[$timestamp] from $source (${body.length} bytes)"
```

## Notes
- An irrefutable destructuring bind — `val Header(timestamp, source) = event.payload.header` — throws `MatchError` at runtime if the pattern can actually fail; it is only safe for product types (case classes, tuples) whose shape is guaranteed to match, not for one variant of a sealed hierarchy that might be a different case.
- For a value that might be one of several sealed variants, use a real `match` with a case per variant rather than an irrefutable bind — the irrefutable form has no fallback for the cases it doesn't expect.
- A `for`-comprehension generator with a pattern (`for Person(name, age) <- people`) silently filters out elements that don't match, rather than raising an error — a useful behavior when that is genuinely intended (extracting one shape out of a mixed collection), but a surprising one if a reader expects every element to be included.
- `pattern-matching-guard-clauses` covers adding a condition alongside a destructuring pattern in the same `case`.

## References
- [Scala 3 Reference — Pattern Matching](https://docs.scala-lang.org/scala3/reference/changed-features/pattern-matching.html)
