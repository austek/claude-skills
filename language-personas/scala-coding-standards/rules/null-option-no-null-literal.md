---
title: Never Use null as a Value
impact: CRITICAL
impactDescription: removes NullPointerException as a possible runtime failure in Scala-authored code
tags: [null-safety, option, nullability]
---

# Never Use null as a Value [CRITICAL]

## Description
`null` exists in Scala only because the language runs on the JVM and must interoperate with Java, which allows it in the type of every `AnyRef` subtype. Idiomatic Scala code never introduces a `null` itself — no method returns `null` to signal absence, no field or local is assigned `null`, and no API accepts `null` as a meaningful argument. `null` is a value that inhabits every reference type yet carries no information at the type level: `def find(id: String): User` compiles whether or not the implementation can return `null`, so nothing at the call site forces the caller to consider "what if there's nothing here" — the failure only appears at runtime, as a `NullPointerException`, often far from where the `null` was actually produced.

The correct replacement is `Option[T]` (`null-option-option-for-absence`): a value that may or may not be present goes in the type signature as `Option[T]`, and the compiler then requires the caller to handle both cases before getting at the value underneath. The one place `null` legitimately appears in Scala code is at the boundary with a Java API that can hand back `null` — wrap that call immediately with `Option(...)`, which maps a `null` result to `None` and a non-null result to `Some`, and let `null` go no further into the codebase than that single conversion point.

## Bad Example
```scala
def findUser(id: String): User =
  if id == "u-1" then User("u-1", "Ada") else null // caller has no signal this can happen

val user = findUser("missing")
println(user.name) // NullPointerException, far from where null originated
```

## Good Example
```scala
def findUser(id: String): Option[User] =
  if id == "u-1" then Some(User("u-1", "Ada")) else None

// Java interop boundary: wrap immediately, let null go no further
def lookupLegacy(id: String): Option[User] =
  Option(legacyJavaUserDao.find(id)) // null -> None automatically

findUser("missing").fold(println("no such user"))(u => println(u.name))
```

## Notes
- `val x: String = null` compiles without a warning in default Scala 3 — `Null` is a subtype of every `AnyRef`, so nothing at the type level stops this; discipline (and linting) is what prevents it, not the type checker by default.
- Scala 3's experimental explicit-nulls mode (`-Yexplicit-nulls`) removes `Null` from the type hierarchy of ordinary reference types and requires `T | Null` to admit `null` explicitly — worth enabling on new projects, but treat it as reinforcement, not a substitute for never writing `null` by hand.
- `Option(javaNullableCall())` is the idiomatic, one-line wrapper for a Java API boundary; do not write `if (x == null) None else Some(x)` by hand when `Option(x)` already does exactly that.
- `null-option-avoid-option-get` covers the parallel mistake of reintroducing an unchecked runtime failure after correctly wrapping a value in `Option`.

## References
- [Scala 3 Reference — Explicit Nulls](https://docs.scala-lang.org/scala3/reference/other-new-features/explicit-nulls.html)
