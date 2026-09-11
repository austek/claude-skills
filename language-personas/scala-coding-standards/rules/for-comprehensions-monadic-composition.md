---
title: Use For-Comprehensions to Compose Monadic Operations
impact: MEDIUM
impactDescription: one composition syntax across Option, Either, List, Future, and IO — no per-type API to relearn
tags: [for-comprehension, monadic-composition, scala3]
---

# Use For-Comprehensions to Compose Monadic Operations [MEDIUM]

## Description
A `for`-comprehension is syntactic sugar, not a language feature tied to a fixed set of types: the compiler rewrites every generator (`x <- expr`) except the last into a `flatMap`, rewrites the last one into a `map`, and rewrites any guard (`if cond`) into a `withFilter` call on the preceding generator's value. Nothing about that desugaring requires a formal `Monad` type class — any type that defines `map` and `flatMap` with compatible signatures qualifies, which is why the identical `for ... yield` syntax composes `Option`, `Either`, `List`, `Try`, `Future`, and effect types like cats-effect's `IO` without switching notation between them.

That uniformity is the payoff: once a sequence of dependent steps is expressed as a `for`-comprehension, a reader familiar with the syntax can follow the data flow top-to-bottom regardless of which of those types is involved, instead of learning each type's chaining idiom separately. Writing the same composition by hand with explicit `flatMap`/`map` calls is equivalent at runtime — the compiler produces the same calls either way — but nests one closure inside another, which reads backward from how the steps actually execute.

Reach for a `for`-comprehension whenever a computation has two or more dependent steps over the same effect type, where a later step needs the value a prior step produced. A single step, or two independent values combined with no dependency between them, usually reads better as a direct `.map`/`.flatMap`/`mapN`-style call — see `for-comprehensions-readability-over-cleverness` for that boundary, and `for-comprehensions-avoid-nested-flatmap` for the specific anti-pattern of hand-nesting three or more dependent steps instead of using this sugar.

## Bad Example
```scala
def shippingQuote(userId: String): Option[BigDecimal] =
  findAddress(userId).flatMap { address =>
    findWeightKg(userId).map { weight =>
      address.zone.ratePerKg * weight
    }
  }
```

## Good Example
```scala
def shippingQuote(userId: String): Option[BigDecimal] =
  for
    address <- findAddress(userId)
    weight  <- findWeightKg(userId)
  yield address.zone.ratePerKg * weight
```

## Notes
- The desugaring is mechanical: every generator but the last becomes `flatMap`, the last becomes `map`, and each guard `if` becomes `withFilter` — knowing this explains any type error a `for`-comprehension produces, since the error is really coming from `flatMap`/`map`/`withFilter` on the underlying type.
- A type needs no inheritance relationship or marker trait to work in a `for`-comprehension — only matching `map`/`flatMap` method signatures — so cats-effect's `IO` and similar third-party effect types compose the same way `Option` and `List` do.
- All generators in one `for`-comprehension must resolve to the same type constructor; mixing `Option` and `List` directly does not type-check without a monad transformer, which is outside the scope of this rule.
- `for-comprehensions-early-exit-with-either` covers the specific, common case of composing `Either`-returning domain steps this way.

## References
- [Scala 3 Book — For Expressions](https://docs.scala-lang.org/scala3/book/control-structures.html#for-expressions)
