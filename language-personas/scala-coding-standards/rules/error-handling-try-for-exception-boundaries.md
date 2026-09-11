---
title: Wrap Exception-Throwing Calls in Try at the Boundary
impact: HIGH
impactDescription: confines try/catch to a single conversion point instead of scattering it through logic
tags: [error-handling, try, exceptions, interop]
---

# Wrap Exception-Throwing Calls in Try at the Boundary [HIGH]

## Description
Some code Scala calls into is always going to throw rather than return a value on failure — Java libraries, `String.toInt`, file and network APIs, most of the JDK. `scala.util.Try` exists specifically to capture that at the exact point of the call: `Try(riskyCall())` runs the block and produces `Success(value)` if it completed normally or `Failure(exception)` if it threw, turning a thrown exception into an ordinary value the rest of the program can pattern-match on, `map` over, or convert. The discipline this rule asks for is narrow but important: wrap the exception-throwing call in `Try` immediately, at the boundary where it happens, rather than letting a raw `try`/`catch` (or an un-caught propagating exception) spread through several layers of business logic.

`Try` only catches non-fatal exceptions, per `scala.util.control.NonFatal` — things like `OutOfMemoryError`, `InterruptedException`, and other JVM-fatal conditions still propagate through it, which is the correct behavior; those are not recoverable business outcomes. Once a `Try` exists, convert it quickly rather than threading it deep into domain logic: `.toEither` turns it into `Either[Throwable, A]` (pair with `.left.map` to translate the raw `Throwable` into a typed domain error, per `error-handling-typed-errors-over-strings`), and `.toOption` discards the exception entirely when only presence/absence matters. `Try`'s own error channel is an untyped `Throwable`, which is precise enough for a one-line interop wrapper but too broad for reasoning about domain failures further inside the codebase.

## Bad Example
```scala
def parseConfig(raw: String): Config =
  try
    val json = legacyJsonParser.parse(raw) // throws on malformed input
    Config.fromJson(json)                  // also throws on missing fields
  catch
    case e: JsonParseException => throw new ConfigException("bad config", e)
    case e: NoSuchElementException => throw new ConfigException("missing field", e)
// try/catch logic is now tangled with the parsing steps it wraps
```

## Good Example
```scala
enum ConfigError:
  case MalformedJson(cause: Throwable)
  case MissingField(cause: Throwable)

def parseConfig(raw: String): Either[ConfigError, Config] =
  Try(legacyJsonParser.parse(raw)).toEither.left.map(ConfigError.MalformedJson.apply)
    .flatMap(json => Try(Config.fromJson(json)).toEither.left.map(ConfigError.MissingField.apply))
```

## Notes
- `Try` catches only `NonFatal` exceptions — JVM-fatal errors (`OutOfMemoryError`, `StackOverflowError`, `InterruptedException`, etc.) still propagate, and that is correct: they are not recoverable outcomes to model as data.
- Convert a `Try` to `Either`/`Option` promptly rather than passing `Try` values deep into business logic — its `Throwable` error channel is too broad for typed, exhaustive handling further downstream.
- `Try` is the right tool specifically for calls that are inherently exception-based (legacy code, Java interop, parsing); it is not a general error-modeling type for new Scala APIs — those should return `Either` directly (`error-handling-either-for-domain-errors`).
- `.recover`/`.recoverWith` on `Try` mirror `Either`'s `.orElse`/`.left.map` for supplying a fallback or transforming the failure without leaving the `Try` type.

## References
- [Scala 3 Standard Library — Try](https://www.scala-lang.org/api/current/scala/util/Try.html)
- [scala.util.control.NonFatal](https://www.scala-lang.org/api/current/scala/util/control/NonFatal$.html)
