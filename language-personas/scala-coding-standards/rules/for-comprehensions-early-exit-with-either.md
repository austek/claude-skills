---
title: Use For-Comprehensions Over Either to Short-Circuit a Validation Pipeline
impact: HIGH
impactDescription: stops at the first failure with zero manual propagation code, and the compiler forces every step's error into one type
tags: [for-comprehension, either, validation]
---

# Use For-Comprehensions Over Either to Short-Circuit a Validation Pipeline [HIGH]

## Description
A multi-step domain pipeline — validate input, look up a related entity, apply a business rule — where any step's failure should abort the remaining steps maps directly onto `Either`'s short-circuiting `flatMap`: a `for`-comprehension over `Either[E, A]` evaluates each generator in order and, the moment one produces a `Left`, skips every remaining line and returns that `Left` as the whole expression's result. This reproduces the control-flow shape of imperative early-return code — stop at the first failure, skip the rest — without a `return` statement (discouraged by house style, and unusable from inside a `for`-comprehension's generated lambdas regardless), without a mutable "has failed" flag, and without hand-written `match` nesting to check the accumulated result after each step.

The precondition is that every step's error type unifies to the same `E` used by the whole `for`-comprehension. A single sealed error type for the pipeline (`error-handling-typed-errors-over-strings`) makes this automatic — each step returns `Either[PipelineError, _]` with a case of `PipelineError` naming its own failure, so the compiler infers one consistent `Either[PipelineError, _]` on every line and the final `yield` result. This rule is the compositional half of `error-handling-either-for-domain-errors`: that rule is about shaping one operation's return type as `Either[E, A]`; this one is about chaining several such operations together.

Reach for something else when steps are independent rather than sequential — a `for`-comprehension stops at the first failure and never runs (or reports) the second and third. Independent validations that should all run and report together (e.g., collecting every invalid form field at once) need an accumulating type instead, which is outside the scope of this rule.

## Bad Example
```scala
enum SignupError:
  case InvalidEmail(raw: String)
  case UsernameTaken(name: String)

def register(email: String, username: String): Either[SignupError, Account] =
  validateEmail(email) match
    case Left(err) => Left(err)
    case Right(validEmail) =>
      checkUsernameFree(username) match
        case Left(err)       => Left(err)
        case Right(validName) => Right(Account(validEmail, validName))
```

## Good Example
```scala
enum SignupError:
  case InvalidEmail(raw: String)
  case UsernameTaken(name: String)

def register(email: String, username: String): Either[SignupError, Account] =
  for
    validEmail <- validateEmail(email)
    validName  <- checkUsernameFree(username)
  yield Account(validEmail, validName)
```

## Notes
- Every generator must share the same left type; widen mismatched error types to a common sealed supertype before composing steps pulled from different subsystems.
- No `return` appears in either form — the `for`-comprehension already produces the "stop at first failure" shape as an expression.
- Reach for Cats' `Validated` (or a hand-rolled accumulator) instead of a `for`-comprehension when independent validations must all run and report together rather than stop at the first failure.
- `for-comprehensions-monadic-composition` covers the general desugaring this relies on; `error-handling-either-for-domain-errors` covers designing the `E` type each step returns.

## References
- [Scala 3 Standard Library — Either](https://www.scala-lang.org/api/current/scala/util/Either.html)
