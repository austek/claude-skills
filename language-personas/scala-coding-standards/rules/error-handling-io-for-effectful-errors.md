---
title: Represent Effectful, Failable Work with IO
impact: HIGH
impactDescription: separates describing an effect from running it, and gives failure a typed handling API
tags: [error-handling, io, effect-system, cats-effect]
---

# Represent Effectful, Failable Work with IO [HIGH]

## Description
An effect type — `cats-effect`'s `IO[A]` or ZIO's `Task[A]` — represents a *description* of a computation that performs side effects and produces an `A`, or fails, without running anything at the point where the value is constructed. `IO.blocking(db.findUser(id))` builds a value describing "go read a user from the database"; nothing happens until something later calls `.unsafeRunSync()` (or the runtime's equivalent) at the program's entry point. This inversion is what lets effectful, failable code stay composable and testable the same way pure functions are: two `IO` values can be combined with `map`/`flatMap`/a `for`-comprehension into a larger `IO` describing a bigger effect, entirely without triggering any actual I/O, and the resulting description can be passed around, logged, or retried as an ordinary value.

Failure inside an `IO` is not a thrown exception interrupting a stack — it's carried in `IO`'s own error channel. `IO.raiseError` constructs a failed description without executing anything, `.attempt` turns `IO[A]` into `IO[Either[Throwable, A]]` so failure becomes an inspectable value, and `.handleErrorWith` supplies a recovery `IO` in place of a failure. This gives effectful code the same explicit, typed handling that `Either` gives pure code — the difference is that `IO` additionally tracks *when* the effect runs, so error handling composes correctly across asynchronous and concurrent boundaries where a plain `try`/`catch` cannot reach.

## Bad Example
```scala
class UserService(db: Database):
  def fetchUser(id: UserId): User =
    db.findUser(id) match // runs immediately, wherever this method happens to be called
      case Some(user) => user
      case None       => throw new NoSuchElementException(s"no user $id")
```

## Good Example
```scala
import cats.effect.IO

class UserService(db: Database):
  def fetchUser(id: UserId): IO[User] =
    IO.blocking(db.findUser(id)).flatMap {
      case Some(user) => IO.pure(user)
      case None       => IO.raiseError(new NoSuchElementException(s"no user $id"))
    }

// composed, still not run
val program: IO[String] =
  userService.fetchUser(id).attempt.map {
    case Right(user) => s"found ${user.name}"
    case Left(_)      => "user not found"
  }

// run exactly once, at the edge of the program
program.unsafeRunSync()
```

## Notes
- `IO` requires a library — `cats-effect` or ZIO — it is not part of the Scala standard library; choose one per project and use it consistently rather than mixing effect systems.
- Keep effect execution ("the end of the world") at the program's entry point or a small number of well-known boundaries; business logic should build and return `IO` values, not call `unsafeRunSync`/`unsafeRunAsync` internally.
- `.attempt` is the `IO` equivalent of converting a `Try` to an `Either` — use it at the point where a failure needs to become an inspectable value rather than continuing to propagate through the effect's own error channel.
- `error-handling-either-for-domain-errors` covers the synchronous, non-effectful case; reach for `IO` specifically when the operation is genuinely effectful (I/O, concurrency, time) and not just "might fail."

## References
- [cats-effect — IO](https://typelevel.org/cats-effect/docs/getting-started)
