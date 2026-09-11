---
title: Enforce Invariants With a Smart Constructor, Not a Public Constructor
impact: HIGH
impactDescription: makes constructing an invalid value a compile-time impossibility instead of a runtime bug
tags: [api-design, smart-constructors, validation, invariants]
---

# Enforce Invariants With a Smart Constructor, Not a Public Constructor [HIGH]

## Description
A type is only as trustworthy as its construction path: `final case class Email(value: String)` lets any caller write `Email("not-an-email")` and get back a perfectly well-typed, perfectly invalid value. Every function that later receives an `Email` then has to choose between trusting a type that doesn't actually guarantee anything, or re-validating the string itself — which defeats the point of having a dedicated type at all. A smart constructor closes that gap: make the primary constructor `private` so `new Email(...)` (and the case class's auto-generated `apply`) is unreachable from outside the type, and expose a companion method that validates the input and returns `Either[E, Email]` (`error-handling-either-for-domain-errors`) — `Left` for invalid input, `Right(email)` once the invariant genuinely holds.

Once construction is gated this way, holding a value of type `Email` anywhere in the codebase is itself proof the invariant was checked — there is no second, unvalidated way to produce one. This is what makes the type worth having: a function taking an `Email` parameter never needs to re-validate it, unlike a function taking a raw `String`. Reserve this pattern for types with a genuine invariant to enforce (well-formed input, a bounded numeric range, a non-empty collection) — a case class with no constraints on its fields gets nothing from a private constructor and just adds ceremony.

## Bad Example
```scala
final case class Email(value: String)

Email("not-an-email") // compiles: nothing enforces the invariant
```

## Good Example
```scala
final case class Email private (value: String):
  def domain: String = value.split("@").last

object Email:
  def parse(raw: String): Either[String, Email] =
    Either.cond(raw.contains("@"), new Email(raw), s"invalid email: $raw")

Email.parse("not-an-email") // Left("invalid email: not-an-email")
Email.parse("ada@example.com") // Right(Email(ada@example.com))
```

## Notes
- Marking a case class's primary constructor `private` also privatizes its compiler-generated companion `apply` — `Email(raw)` from outside the type no longer compiles, only `Email.parse(raw)` does.
- The compiler-generated `copy` method stays public regardless — `email.copy(value = "invalid")` still compiles and bypasses `parse` entirely; if that's a real risk for a given type, define a private `copy` explicitly in the class body (the compiler skips generating one once you've written your own).
- `api-design-opaque-types-for-domain-primitives` covers giving the wrapped value a distinct, zero-cost type; combine the two when a primitive needs both nominal distinctness and a validated invariant.
- Prefer this over throwing from a public constructor (`error-handling-no-throw-for-control-flow`) — an invalid `Email` should be an ordinary `Left`, not an exception the caller has to know to catch.

## References
- [Scala 3 Book — Case Classes](https://docs.scala-lang.org/scala3/book/domain-modeling-tools.html#case-classes)
