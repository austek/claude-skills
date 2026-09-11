---
title: Require an ExecutionContext Explicitly, Don't Import a Default One
impact: MEDIUM
impactDescription: lets a caller choose the right thread pool for the work, instead of every method silently picking one for them
tags: [concurrency, execution-context, future]
---

# Require an ExecutionContext Explicitly, Don't Import a Default One [MEDIUM]

## Description
`scala.concurrent.ExecutionContext.Implicits.global` is a fixed, program-wide thread pool sized for CPU-bound work, and importing it inside a library method bakes that specific scheduling choice into every caller of that method, permanently and invisibly. A method that does `import scala.concurrent.ExecutionContext.Implicits.global` and then creates a `Future` has decided, on behalf of every caller, that the global pool is the right place to run — even for a caller that's already managing its own pool, running inside a framework that provides one (an HTTP server's request-handling context, an actor system's dispatcher), or specifically routing blocking work to a separate blocking pool (`effect-systems-no-blocking-inside-effect` covers the same pool-isolation concern for `IO`). Worse, mixing genuinely blocking calls into `Future`s scheduled on the global pool can starve it for unrelated concurrent work across the whole program, since the global pool has no separate blocking-safe lane the way cats-effect's runtime does.

The fix is mechanical: accept the `ExecutionContext` as an explicit `using` parameter instead of importing one, and let it flow in from whoever is actually positioned to decide which pool is appropriate — usually the application's entry point or a specific module boundary, not each individual method deep in a call graph. This is the same principle `concurrency-future-vs-io` applies at the level of "pick a different concurrency primitive" — cats-effect's `IO` sidesteps this entirely by not requiring any `ExecutionContext` at the call site at all, deferring the scheduling decision to the runtime that eventually executes the `IO`.

## Bad Example
```scala
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.Future

def fetchAll(ids: List[UserId]): Future[List[User]] =
  Future.traverse(ids)(fetchOne) // always runs on the global pool; no caller can override this
```

## Good Example
```scala
import scala.concurrent.{ExecutionContext, Future}

def fetchAll(ids: List[UserId])(using ec: ExecutionContext): Future[List[User]] =
  Future.traverse(ids)(fetchOne) // the caller decides which pool this runs on
```

## Notes
- `implicits-givens-explicit-imports-no-wildcard` covers importing an *existing* given by name instead of a wildcard; this rule is about not baking a specific `ExecutionContext` choice into a method's implementation at all, regardless of import style.
- A `main`/application-entry-point method (or a test) is a legitimate place to provide `ExecutionContext.Implicits.global` explicitly as the one concrete choice for the whole program — the problem is a library-level method choosing it silently on a caller's behalf.
- `IO`-based code (`concurrency-future-vs-io`) avoids this whole class of decision at the call-site level; this rule matters specifically for code that still legitimately uses `Future`.
- Threading an `ExecutionContext` explicitly also makes it visible in a method's signature that the method is asynchronous and pool-dependent, rather than that fact hiding in an import line.

## References
- [Scala Standard Library — ExecutionContext](https://www.scala-lang.org/api/current/scala/concurrent/ExecutionContext.html)
