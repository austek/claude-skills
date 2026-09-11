---
title: Acquire and Release Resources With Resource, Not Manual try/finally
impact: HIGH
impactDescription: guarantees release runs even on cancellation, not only on a normal return or a thrown exception
tags: [effect-systems, cats-effect, resource-safety]
---

# Acquire and Release Resources With Resource, Not Manual try/finally [HIGH]

## Description
A resource that must be released after use — a file handle, a database connection, a lock — needs its release step to run under every way the code using it can end: a normal return, an exception, and, in a concurrent effect system, cancellation of the fiber holding it. A hand-written `try`/`finally` wrapped around `IO`-returning code only covers the first two cases correctly, because it doesn't account for the block being cancelled mid-flight by another fiber (`IO.race`, a timeout, a parent scope shutting down) before the `finally` clause's own effect gets a chance to run to completion — a `finally` block is itself just more code that has to run to completion to have its intended effect, and cancellation is precisely what can stop code from running to completion.

Cats-effect's `Resource[F, A]` (built on the lower-level `bracket`/`bracketCase` operations) encodes acquire/use/release as a single value that closes this gap: `Resource.make(acquire)(release)` pairs an acquiring `IO[A]` with a releasing `A => IO[Unit]`, and `.use(f)` runs `f` against the acquired resource with the runtime itself guaranteeing `release` runs afterward — on success, on a thrown error, and on cancellation — because the runtime's cancellation handling is aware of resource scopes and runs pending releases as part of unwinding a cancelled fiber, rather than relying on ordinary sequential code reaching a `finally` clause. Multiple `Resource` values compose in a `for`-comprehension, acquiring in order and releasing in reverse order automatically — the same nesting a hand-written pyramid of `try`/`finally` blocks would need to get right manually.

## Bad Example
```scala
import cats.effect.IO

def readFirstLine(path: String): IO[String] =
  IO.blocking {
    val source = scala.io.Source.fromFile(path)
    try source.getLines().next()
    finally source.close() // does not run if this IO is cancelled before reaching this point
  }
```

## Good Example
```scala
import cats.effect.{IO, Resource}

def sourceResource(path: String): Resource[IO, scala.io.Source] =
  Resource.make(IO.blocking(scala.io.Source.fromFile(path)))(source => IO.blocking(source.close()))

def readFirstLine(path: String): IO[String] =
  sourceResource(path).use(source => IO.blocking(source.getLines().next()))
```

## Notes
- `Resource.make(acquire)(release)` is the general constructor; `Resource.fromAutoCloseable(acquireIO)` is a shorthand for anything implementing Java's `AutoCloseable`.
- `.use(f)`'s release guarantee covers cancellation specifically — the property a plain `try`/`finally` wrapped around an `IO` value does not have.
- Composing several `Resource` values in a `for`-comprehension acquires in the written order and releases in the reverse order automatically.
- ZIO's scoped resources (`ZIO.acquireRelease`) and Ox's structured-scope resource helpers solve the same problem for their respective effect models.

## References
- [Cats Effect — Resource](https://typelevel.org/cats-effect/docs/std/resource)
