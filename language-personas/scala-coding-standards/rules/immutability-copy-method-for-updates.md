---
title: Use copy to Produce Updated Instances
impact: HIGH
impactDescription: replaces in-place mutation with explicit, shareable, structurally-safe value updates
tags: [immutability, case-class, copy]
---

# Use copy to Produce Updated Instances [HIGH]

## Description
Every `case class` gets a compiler-generated `copy` method that builds a new instance from an existing one, overriding only the named fields and reusing the rest unchanged. It is the idiomatic replacement for "update in place": instead of a setter mutating a field on a shared instance — which any other holder of that reference would then observe — `copy` produces an independent value, leaving the original and every other reference to it untouched. This is what makes case classes safe to pass around and share: nothing that receives one needs to defend against another part of the program silently changing it later.

`copy` is a shallow operation — it replaces the top-level fields you name and shares the rest of the structure by reference, including nested case classes. Updating a field nested two or more levels deep therefore means chaining `copy` calls inward-out (`outer.copy(inner = outer.inner.copy(field = value))`), which gets noisy past two or three levels of nesting. At that point, reach for a lens library (Monocle is the standard choice in the Scala ecosystem) rather than hand-rolling deeper copy chains — the shallow-update problem `copy` solves for one level is exactly what lenses generalize to arbitrary depth.

## Bad Example
```scala
class MutableOrder(var status: String, var total: BigDecimal)

def markPaid(order: MutableOrder): Unit =
  order.status = "paid" // mutates the caller's instance in place

val order = MutableOrder("pending", BigDecimal(50))
markPaid(order)
// every other holder of `order` now sees "paid" too, whether that was intended or not
```

## Good Example
```scala
case class Order(status: String, total: BigDecimal)

def markPaid(order: Order): Order =
  order.copy(status = "paid")

val order = Order("pending", BigDecimal(50))
val paidOrder = markPaid(order)
// `order` is unchanged; `paidOrder` is the new value — callers choose which they hold
```

## Notes
- `copy` accepts named arguments only for the fields being changed; every other field keeps its current value automatically — there is no need to repeat the whole constructor call.
- Nested updates: `order.copy(customer = order.customer.copy(email = newEmail))` is fine at one level of nesting; past that, prefer a lens (`Focus[Order](_.customer.address.city).replace(newCity)(order)` with Monocle) over deeper manual chains.
- `copy` performs a shallow copy — if a field holds a mutable collection or mutable object, the new instance shares that same mutable reference; this is another reason field types should themselves be immutable (`immutability-immutable-collections-default`).
- `copy` is generated automatically and does not run through the class's compact constructor validation logic in the same way explicit construction does in some other languages — any invariant checked at construction should be re-checked or structured so `copy` cannot produce an invalid instance (e.g. via a smart constructor rather than relying on the default one alone for validated types).

## References
- [Scala 3 Book — Case Classes](https://docs.scala-lang.org/scala3/book/taste-modeling.html#case-classes)
- [Monocle — Optics Library for Scala](https://www.optics.dev/Monocle/)
