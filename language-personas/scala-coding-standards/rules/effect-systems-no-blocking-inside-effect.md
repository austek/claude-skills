---
title: Never Block a Thread Inside a Non-Blocking Effect
impact: HIGH
impactDescription: prevents thread-pool starvation that silently stalls unrelated concurrent work
tags: [effect-systems, cats-effect, blocking, concurrency]
---

# Never Block a Thread Inside a Non-Blocking Effect [HIGH]

## Description
Cats-effect's default runtime multiplexes many logical `IO` fibers onto a small, fixed-size pool of worker threads (sized to the number of CPU cores), on the assumption that any individual fiber yields control back to the scheduler quickly rather than occupying a worker thread for an extended, uninterruptible stretch. Calling a genuinely blocking operation — a synchronous JDBC query, `Thread.sleep`, a blocking HTTP client call — directly inside an `IO` (e.g. `IO.delay(blockingCall())`) violates that assumption: the worker thread executing that fiber is unavailable to run any other fiber for as long as the blocking call takes, and with enough concurrent blocking calls, all of a runtime's worker threads can end up occupied this way simultaneously — starving completely unrelated `IO` fibers that have nothing to do with the blocking operation and have no way to run until a worker thread frees up.

Cats-effect provides `IO.blocking(blockingCall())` specifically for this case: it wraps the same kind of call, but schedules it on a separate, much larger blocking-specific thread pool rather than the compute pool used for ordinary fibers, so a slow or blocked call there doesn't compete with the rest of the program's concurrent work for the same limited threads. The rule is mechanical: anything that can block the calling thread for I/O, a lock, or a sleep belongs in `IO.blocking`, not plain `IO.delay`/`IO(...)`, which is reserved for effects that return quickly once started.

## Bad Example
```scala
import cats.effect.IO

def fetchUser(id: String): IO[User] =
  IO.delay(jdbcConnection.queryUser(id)) // blocks a compute-pool worker thread for the query's full duration
```

## Good Example
```scala
import cats.effect.IO

def fetchUser(id: String): IO[User] =
  IO.blocking(jdbcConnection.queryUser(id)) // runs on the blocking pool, leaving compute workers free
```

## Notes
- `IO.blocking` is the right tool for synchronous JDBC, blocking file I/O, `Thread.sleep`, and any legacy blocking Java API without a non-blocking equivalent.
- `IO.sleep(duration)` is cats-effect's own non-blocking delay and should always be used over `Thread.sleep` inside an `IO` — it suspends the fiber without occupying any worker thread at all.
- ZIO has the same distinction as `ZIO.attemptBlocking`; Ox's structured forks run on virtual threads, where a blocking call is far cheaper but can still exhaust a bounded carrier-thread pool under enough concurrent load — check the runtime's own docs rather than assuming identical pool-sizing behavior across libraries.
- `effect-systems-resource-safety` covers the related concern of a blocking resource (a connection, a file handle) being released correctly regardless of which pool it ran on.

## References
- [Cats Effect — Thread Model](https://typelevel.org/cats-effect/docs/thread-model)
