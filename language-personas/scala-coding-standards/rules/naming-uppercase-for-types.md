---
title: Use UpperCamelCase for Classes, Traits, Objects, and Type Parameters
impact: LOW
impactDescription: lets a reader tell a type from a value on sight, with no need to check where it was declared
tags: [naming, style, conventions]
---

# Use UpperCamelCase for Classes, Traits, Objects, and Type Parameters [LOW]

## Description
Every type-level name in Scala — a `class`, `trait`, `object`, `enum`, or type alias — takes `UpperCamelCase`: `PaymentGateway`, `OrderService`, `UserId`. This is the direct counterpart to `naming-lowercase-for-values`'s rule for ordinary bindings, and the two conventions together are what let a reader classify any identifier at a glance, purely from its first letter, without needing to check a declaration or an import to know whether `Config` names a type or `config` names a value carrying one. Getting this backwards on a `trait` or `class` doesn't just read oddly — an object extending a lowercase-initial supertype, or a class whose name collides in casing with an intended companion value, invites exactly the kind of ambiguity the convention exists to prevent.

Type parameters follow a narrower convention: a single uppercase letter (`T`, `A`, `K`, `V`, `F[_]`) is idiomatic for a type parameter with no more specific role than "some type" — `List[T]`, `Map[K, V]`, `def traverse[F[_], A, B](...)`. Once a type parameter has an actual constraint or meaning worth naming (`Encoder[Domain]`'s `Domain`, or a self-type-like bound), a short, descriptive `UpperCamelCase` name reads better than a bare letter picked from context alone — but it still follows the same uppercase-initial convention every other type does, never a lowercase one.

## Bad Example
```scala
trait paymentGateway:
  def charge(amount: BigDecimal): Unit

class orderService(gateway: paymentGateway)
```

## Good Example
```scala
trait PaymentGateway:
  def charge(amount: BigDecimal): Unit

class OrderService(gateway: PaymentGateway)
```

## Notes
- `naming-lowercase-for-values` covers the mirror-image convention for values, methods, and parameters — read together, the two rules make the first letter of any identifier tell you which kind of thing it names.
- Single-letter type parameters (`T`, `A`, `F[_]`) are conventional for generic, unconstrained positions; reserve a descriptive name for a type parameter whose role in the signature isn't obvious from context alone.
- Package names are the one place Scala convention runs the other way — all-lowercase, no camelCase — following the same convention Java uses.
- `implicits-givens-type-class-pattern` and `implicits-givens-context-bounds-syntax` both lean on this convention for their type parameters (`JsonEncoder[T]`, `[T: Ordering]`) reading naturally as types.

## References
- [Scala Style Guide — Naming Conventions](https://docs.scala-lang.org/style/naming-conventions.html)
