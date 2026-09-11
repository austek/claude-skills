---
title: Never Use throw for Ordinary Control Flow
impact: CRITICAL
impactDescription: keeps expected outcomes visible in the type signature instead of hidden in a stack unwind
tags: [error-handling, exceptions, control-flow, anti-pattern]
---

# Never Use throw for Ordinary Control Flow [CRITICAL]

## Description
Reserve `throw` for conditions that are genuinely exceptional — a programmer error, a broken invariant, a failure so far outside the method's contract that no caller could reasonably be expected to handle it as a normal branch. "User not found," "validation failed," "the requested page doesn't exist" are not exceptional in this sense: they are routine, expected outcomes of calling the method, and a caller is meant to branch on them as part of ordinary logic. Throwing for these discards the same information `Either`/`Option` are built to preserve — the JVM does not force callers to handle unchecked exceptions, so nothing at the call site signals that a particular outcome is possible, and the eventual `catch` block ends up living far from the `throw`, with no compiler-checked link between them.

Beyond the missing type signal, `throw` breaks the equational reasoning a function's return value is supposed to provide: two functions with the same input can differ in whether they return a value or unwind the stack, and that difference is invisible until runtime. It's also measurably more expensive than an ordinary return — the JVM captures a stack trace at `throw` time by default, work that is wasted for something that should have been a plain conditional. Model the same situation as `Either[E, A]` (`error-handling-either-for-domain-errors`) or `Option[A]` and let the caller branch on the return value the same way it would branch on any other data.

## Bad Example
```scala
class UserRepository:
  def findById(id: UserId): User =
    users.get(id) match
      case Some(user) => user
      case None       => throw new NoSuchElementException(s"user $id not found")

// caller treats a routine "not found" as if it were exceptional
try
  val user = repository.findById(id)
  renderProfile(user)
catch
  case _: NoSuchElementException => renderNotFound()
```

## Good Example
```scala
class UserRepository:
  def findById(id: UserId): Option[User] =
    users.get(id)

repository.findById(id).fold(renderNotFound())(renderProfile)
```

## Notes
- `throw` is still correct for what it names: a violated invariant, a bug in the caller's own code (an assertion failing), or a condition so unexpected that no meaningful typed recovery exists — these should crash loudly rather than be quietly modeled as just another `Left`.
- This mirrors the workspace-wide rule against `return`: both are early-exit control flow that bypasses the function's own type as the single source of truth for what it produces.
- Catching an exception immediately at the boundary where a throwing API is unavoidable (Java interop, a parsing library) and converting it to `Either`/`Option` right there is not a violation of this rule — see `error-handling-try-for-exception-boundaries`; the rule targets *originating* a `throw` for an expected outcome, not receiving one from code outside your control.
- `error-handling-typed-errors-over-strings` and `pattern-matching-sealed-trait-exhaustiveness` cover how to make the replacement `Either`/`Option` structure exhaustively handled at the call site, so nothing is lost by not throwing.

## References
- [Scala 3 Book — Control Structures](https://docs.scala-lang.org/scala3/book/control-structures.html)
