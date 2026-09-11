---
title: Chain orElse Instead of Nesting match on Option
impact: MEDIUM
impactDescription: turns a pyramid of nested fallback checks into one flat, readable pipeline
tags: [option, pattern-matching, readability]
---

# Chain orElse Instead of Nesting match on Option [MEDIUM]

## Description
When a value can come from any of several optional sources tried in priority order — a request header, then a query parameter, then a config default — the naive way to express that is a `match` on the first `Option`, with a nested `match` on the second inside the `None` branch, and a further nested `match` inside that one, and so on for every fallback. Each additional source adds another level of indentation and pushes the eventual default further right, until the actual precedence order is harder to read out of the nesting than it would be from a flat list.

`Option.orElse` expresses exactly this shape directly: `opt1.orElse(opt2).orElse(opt3)` evaluates to the first `Some` among `opt1`, `opt2`, `opt3`, in order, with no nesting at all — the fallback chain reads top-to-bottom as the same priority order it represents. `orElse`'s argument is by-name, so `opt2` (and each subsequent fallback) is only evaluated if the preceding option was `None`, which matters when computing a fallback candidate is itself expensive — a network call, a file read — and not just the wrapping of an already-known value. Terminate the chain with `.getOrElse(default)` to fall through to a concrete, always-available value once every optional source has been exhausted.

## Bad Example
```scala
def resolveTimeout(header: Option[Int], query: Option[Int], config: Option[Int]): Int =
  header match
    case Some(h) => h
    case None =>
      query match
        case Some(q) => q
        case None =>
          config match
            case Some(c) => c
            case None    => defaultTimeoutSeconds
```

## Good Example
```scala
def resolveTimeout(header: Option[Int], query: Option[Int], config: Option[Int]): Int =
  header
    .orElse(query)
    .orElse(config)
    .getOrElse(defaultTimeoutSeconds)
```

## Notes
- `orElse` takes a by-name `Option[T]` parameter, so later fallbacks in the chain are not computed unless an earlier one was `None` — this is not just a style improvement, it can avoid real work.
- `getOrElse` is also by-name and belongs at the end of a chain to unwrap to a plain value; do not follow it with `.get` — see `null-option-avoid-option-get`.
- A `match` is still the better tool when the branches need materially different logic rather than just picking among equivalent alternative sources — reach for `orElse` specifically for the "same kind of value from several possible places" shape.
- The same chaining shape applies to `Either` fallbacks via `.orElse` on `Either`, which keeps whichever side's error information is most relevant when every alternative fails.

## References
- [Scala 3 Standard Library — Option.orElse](https://www.scala-lang.org/api/current/scala/Option.html#orElse)
