---
title: Bound Every Concurrent Fork to a Structured Scope
impact: HIGH
impactDescription: guarantees a background computation is always joined or cancelled before its caller returns, never left running unsupervised
tags: [concurrency, ox, structured-concurrency, cancellation]
---

# Bound Every Concurrent Fork to a Structured Scope [HIGH]

## Description
Unstructured concurrency — a raw `Thread`, or a `Future` created and never awaited — has no defined relationship between the concurrent work and the code that started it: the launching method can return, throw, or otherwise finish while the background computation keeps running, unobserved, with its eventual success or failure going nowhere. A failure on the "main" path doesn't stop it either — a `Future` fired off for a side task keeps executing even if the code that fired it fails immediately afterward, because nothing connects the two lifetimes. This is a common source of orphaned work and silently swallowed failures, and it gets worse, not better, as more concurrent tasks are added, because each one needs its own ad hoc tracking to avoid the same problem.

Ox's `supervised { ... }` scope makes the relationship structural instead of ad hoc: every `fork { ... }` started inside the block is owned by that scope, and `supervised` does not return until every fork it owns has completed — joined if it succeeded, cancelled if the scope is unwinding because of a failure elsewhere. This holds even for a fork the body never explicitly calls `.join()` on: the scope still waits for it (or cancels it) before letting control leave the block, so forgetting to join a fork is a missed result, not a leaked thread. And because failure and cancellation are scope-wide, one fork throwing cancels its siblings automatically — a caller working inside a `supervised` block gets fail-fast semantics for free, instead of having to build them by hand across independent `Future`s.

## Bad Example
```scala
import scala.concurrent.{ExecutionContext, Future}

def fetchDashboard()(using ec: ExecutionContext): Stats =
  Future { fetchAlerts() } // fire-and-forget: never awaited, never joined, its failure never observed here
  fetchStats() // if this throws, the alerts Future above keeps running regardless
```

## Good Example
```scala
import ox.{supervised, fork}

def fetchDashboard(): Stats =
  supervised {
    fork { fetchAlerts() } // never explicitly joined, but supervised still waits for it (or cancels it) before returning
    fetchStats()            // if this throws, the alerts fork is cancelled as part of unwinding the scope
  }
```

## Notes
- `effect-systems-direct-style-ox` covers Ox's direct-style programming model itself (no monadic wrapper); this rule covers the safety property — bounded fork lifetime and scope-wide failure propagation — that structured concurrency buys regardless of which effect model a codebase uses.
- Cats-effect and ZIO give the same guarantee monadically, through their own structured combinators (`IO.race`, `IO.both`, `parTraverse`, fiber supervision) — the "structured" property is not unique to Ox, only its direct-style syntax is.
- A fork's result still needs `.join()` to actually retrieve it; the scope's own wait-for-completion guarantee is about lifetime and cancellation, not about handing the value back to the caller.
- `concurrency-avoid-shared-mutable-state` matters even more inside a `supervised` block with multiple forks — structuring lifetimes doesn't protect state those forks mutate concurrently.

## References
- [Ox — Structured Concurrency](https://ox.softwaremill.com/latest/structured-concurrency.html)
