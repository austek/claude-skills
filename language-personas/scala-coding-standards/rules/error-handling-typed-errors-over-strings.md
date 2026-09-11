---
title: Model Errors as Typed ADTs, Not Strings
impact: HIGH
impactDescription: lets callers pattern-match exhaustively on failure instead of parsing message text
tags: [error-handling, adt, enum, either]
---

# Model Errors as Typed ADTs, Not Strings [HIGH]

## Description
The error side of an `Either[E, A]` (or an effect's failure channel) is only as useful as `E` is specific. `Either[String, User]` tells a caller that *something* went wrong and gives a human-readable sentence about it, but nothing about how many distinct failure modes exist or how to distinguish one from another except by inspecting the string's contents — which is brittle, breaks silently if the message wording changes, and gives the compiler nothing to check. A closed set of typed error cases, defined with a Scala 3 `enum` (or a `sealed trait` hierarchy when cases need heterogeneous shapes beyond an enum's uniform constructor list), lets a caller `match` on the error exhaustively and handle each failure mode with logic specific to it — retry a timeout, surface a validation message to the user, escalate an unexpected state — with the compiler flagging any case the `match` forgets.

A typed error case can still carry a human-readable message as one of its fields for logging or display — the point is not to remove message text, but to stop using message text as the *only* way to identify which failure occurred. Keep the error hierarchy scoped to one layer or bounded context rather than one giant application-wide error enum; a `UserError` that only `UserService` produces is easier to keep exhaustive and meaningful than an `AppError` enum that every module contributes cases to.

## Bad Example
```scala
def findUser(id: UserId): Either[String, User] =
  users.get(id).toRight(s"user $id not found")

def validateEmail(email: String): Either[String, String] =
  if email.contains("@") then Right(email) else Left(s"'$email' is not a valid email")

// caller can only branch by string-matching, or not branch at all
```

## Good Example
```scala
enum UserError:
  case NotFound(id: UserId)
  case InvalidEmail(value: String)

def findUser(id: UserId): Either[UserError, User] =
  users.get(id).toRight(UserError.NotFound(id))

def validateEmail(email: String): Either[UserError, String] =
  Either.cond(email.contains("@"), email, UserError.InvalidEmail(email))

def handle(error: UserError): Response = error match
  case UserError.NotFound(id)       => Response.notFound(s"no user $id")
  case UserError.InvalidEmail(raw)  => Response.badRequest(s"invalid email: $raw")
```

## Notes
- A Scala 3 `enum` compiles to a sealed hierarchy under the hood, so it gets the same exhaustiveness checking in `match` as a hand-written `sealed trait` — prefer `enum` for the common case of a closed set of error cases with simple constructors.
- Carrying a `cause: Throwable` field on a case (e.g. when wrapping an interop failure from `error-handling-try-for-exception-boundaries`) is fine — the case itself is still a typed, matchable value, independent of whatever text the wrapped exception happened to have.
- Keep error enums scoped per service or bounded context; a single application-wide error type tends to accumulate unrelated cases and forces every handler to match on failure modes it will never actually see.
- `pattern-matching-sealed-trait-exhaustiveness` covers the general case of exhaustive matching on closed hierarchies, of which a typed error enum is one specific, common application.

## References
- [Scala 3 Reference — Enums](https://docs.scala-lang.org/scala3/reference/enums/enums.html)
