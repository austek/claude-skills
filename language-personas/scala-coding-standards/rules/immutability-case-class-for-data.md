---
title: Model Data with case class, Not Plain Classes
impact: HIGH
impactDescription: gets immutability, structural equality, and pattern-matching support for free
tags: [immutability, case-class, data-modeling]
---

# Model Data with case class, Not Plain Classes [HIGH]

## Description
A `case class` is Scala's dedicated construct for a value that is defined by its data: every constructor parameter is a `val` by default, the compiler generates structural `equals`/`hashCode`/`toString`, a `copy` method for producing updated instances (`immutability-copy-method-for-updates`), an `unapply` extractor so the type works directly in `match` expressions and destructuring binds, and a companion `apply` for construction without `new`. A plain `class` gives none of this automatically — reaching for one to hold data means hand-writing equality and hashing (easy to get wrong or let drift out of sync with the fields), exposing mutable fields unless every one is manually marked `val`, and losing pattern-match support entirely.

Reserve plain `class` for types that are defined by behavior or identity rather than data — a service with dependencies injected through its constructor, a resource wrapping a connection, anything where two instances with identical field values are not meant to be considered equal. If a type's purpose is to carry a fixed set of values around — a domain record, an event, a configuration, a DTO — it is a `case class`. For a closed set of data variants, pair `case class` with `sealed trait` or a Scala 3 `enum` (`pattern-matching-sealed-trait-exhaustiveness`) rather than one case class with optional fields standing in for different shapes.

## Bad Example
```scala
class Order(val id: String, val items: List[String], val total: BigDecimal):
  override def equals(other: Any): Boolean = other match
    case o: Order => o.id == id && o.items == items && o.total == total
    case _        => false
  override def hashCode: Int = (id, items, total).hashCode
  // toString, copy, and pattern-match support still missing
```

## Good Example
```scala
case class Order(id: String, items: List[String], total: BigDecimal)

val order = Order("o-1", List("widget"), BigDecimal(19.99))
val discounted = order.copy(total = order.total * BigDecimal("0.9"))

order match
  case Order(id, _, total) if total > 100 => println(s"$id qualifies for free shipping")
  case Order(id, _, _)                    => println(s"$id standard shipping")
```

## Notes
- Every case class constructor parameter is a `val` unless explicitly declared `var` — never declare one `var`; that reintroduces mutation into a type whose entire contract is value equality (`immutability-val-over-var`).
- Case classes compare structurally, so two `Order` instances with the same field values are `==`, regardless of identity — the opposite default from a plain class, and usually the correct default for data.
- Extending a case class with another case class is disallowed (case-to-case inheritance is forbidden); model variation with a sealed hierarchy of case classes/objects instead.
- A single-field case class wrapping a primitive (an opaque type or a value class) is the idiomatic way to give a bare `String`/`Int` a domain-specific type without runtime overhead.

## References
- [Scala 3 Book — Case Classes](https://docs.scala-lang.org/scala3/book/taste-modeling.html#case-classes)
