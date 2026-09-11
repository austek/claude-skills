---
title: Never Share a var or Mutable Collection Across Concurrent Tasks
impact: CRITICAL
impactDescription: eliminates data races and lost updates instead of leaving them to show up intermittently in production
tags: [concurrency, shared-state, race-conditions, ref]
---

# Never Share a var or Mutable Collection Across Concurrent Tasks [CRITICAL]

## Description
A `var` or a mutable collection (`scala.collection.mutable.Map`, an `Array`) accessed from a single thread is safe by construction — there's only ever one reader and one writer, and they never overlap in time. The instant that same binding is reachable from more than one concurrently running task — parallel `Future` callbacks, several Ox forks in one `supervised` scope, threads sharing an instance — that guarantee disappears: a compound operation like `count += 1` is not atomic, it's a read followed by a write, and two tasks interleaving those steps can both read the same starting value and both write back the same result, silently losing one of the two updates. This class of bug is exactly as dangerous as it is hard to catch: it reproduces intermittently, under load, and a test suite that happens to run everything sequentially will never see it.

The fix is not "synchronize the `var`" — a lock closes the race but reintroduces the coordination cost and deadlock risk concurrency was supposed to avoid, and it's easy to get the locking scope wrong. Model shared mutable state as an explicit, concurrency-safe primitive instead: `cats.effect.Ref[F, A]` turns a mutation into a describable, atomically-applied `F[_]` operation (`.update`, `.updateAndGet`) with no lock for the caller to manage, and the same discipline applies to any effect system's equivalent (Ox's `java.util.concurrent.atomic` types used directly, ZIO's `Ref`). Where the state doesn't need to be shared at all — most of it doesn't — the better fix is to not share it: give each concurrent task its own local accumulator and combine the results once every task has finished.

## Bad Example
```scala
class RequestTracker:
  var processed: Int = 0 // shared across every fiber calling handle concurrently

  def handle(req: Request)(using ec: ExecutionContext): Future[Unit] =
    doWork(req).map(_ => processed += 1) // not atomic: concurrent callbacks can lose updates
```

## Good Example
```scala
import cats.effect.{IO, Ref}
import cats.syntax.parallel.*

def processAll(requests: List[Request], processed: Ref[IO, Int]): IO[Unit] =
  requests.parTraverse(req => handle(req) *> processed.update(_ + 1)).void
```

## Notes
- `Ref[F, A]`'s update methods (`.update`, `.updateAndGet`, `.modify`) apply the given function atomically, with no possibility of two concurrent callers interleaving a read and a write the way a raw `var` allows.
- `anti-patterns-no-var-escape` covers the narrower case of a single `var` leaking out of its defining scope; this rule covers the specific, more severe consequence once that escaped `var` is also reachable from concurrent tasks.
- Preferring immutable data and passing results back through return values (rather than through a shared mutable accumulator) avoids needing `Ref` at all for most concurrent fan-out work — reach for `Ref` when genuinely shared, mutable, cross-task state is unavoidable.
- `effect-systems-referential-transparency`'s `Ref` example covers the same primitive from the angle of keeping a function pure; this rule covers it from the angle of concurrent-access safety.

## References
- [Cats Effect — Ref](https://typelevel.org/cats-effect/docs/std/ref)
