---
title: Use Either for Operations That Can Fail With a Reason
impact: HIGH
impactDescription: carries a specific failure reason through the type instead of discarding it
tags: [either, error-handling, nullability]
---

# Use Either for Operations That Can Fail With a Reason [HIGH]

## Description
`Option[T]` answers "is there a value or not" but throws away any information about *why* not — every `None` looks the same, whether the cause was a missing field, an invalid format, or a failed business rule. When a caller needs to know the specific reason an operation didn't produce a value — to show a targeted error message, to decide whether to retry, to log something actionable — model the result as `Either[E, A]` instead. By convention `Left` holds the failure and `Right` holds the success value; since Scala 2.12, `Either` is right-biased, so `map`/`flatMap` operate on the `Right` case directly without needing a `.right` projection, which lets `Either`-returning steps chain in a `for`-comprehension exactly like `Option` does, short-circuiting on the first `Left`.

Reach for `Either` specifically when the failure is expected and the caller is meant to branch on it as part of normal control flow — parsing input, validating a request, a business rule rejecting a transition. It is not a general substitute for `Option` (still the right choice for plain "value or nothing" with no error detail to carry) or for exceptions (still appropriate for truly exceptional, non-recoverable conditions — see `error-handling-no-throw-for-control-flow`). Keep the error type specific rather than a bare `String`; `null-option-either-for-expected-failure` composes naturally with a small sealed error hierarchy (`error-handling-typed-errors-over-strings`) so callers can pattern-match on exactly which failure occurred.

## Bad Example
```scala
def parseAge(input: String): Option[Int] =
  input.toIntOption.filter(_ >= 0)
  // caller sees None whether the input wasn't a number, was negative, or was empty — no way to tell which
```

## Good Example
```scala
enum AgeError:
  case NotANumber(input: String)
  case Negative(value: Int)

def parseAge(input: String): Either[AgeError, Int] =
  input.toIntOption match
    case None                    => Left(AgeError.NotANumber(input))
    case Some(n) if n < 0        => Left(AgeError.Negative(n))
    case Some(n)                 => Right(n)

def describe(input: String): String =
  parseAge(input).fold(
    {
      case AgeError.NotANumber(raw) => s"'$raw' is not a number"
      case AgeError.Negative(n)     => s"$n cannot be negative"
    },
    age => s"age is $age"
  )
```

## Notes
- `Either`'s right-bias (Scala 2.12+ and Scala 3) means `either.map(f)`/`either.flatMap(f)` act on `Right` directly; use `.left.map(f)` to transform the error channel specifically.
- `for`-comprehensions over `Either[E, A]` work the same way they do over `Option[A]`, and short-circuit to the first `Left` encountered.
- Do not mix signaling strategies for the same failure category within one call chain — a method returning `Either[E, A]` should not also throw for the same class of expected failure.
- `null-option-option-for-absence` covers the simpler "value or nothing" case where no failure reason needs to be carried.

## References
- [Scala 3 Standard Library — Either](https://www.scala-lang.org/api/current/scala/util/Either.html)
