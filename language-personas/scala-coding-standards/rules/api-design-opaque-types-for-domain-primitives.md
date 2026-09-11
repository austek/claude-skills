---
title: Wrap Domain Primitives in Opaque Types, Not Raw String/Long/Int
impact: HIGH
impactDescription: turns "passed the wrong id" from a runtime bug into a compile error, at zero runtime cost
tags: [api-design, opaque-types, type-safety, scala3]
---

# Wrap Domain Primitives in Opaque Types, Not Raw String/Long/Int [HIGH]

## Description
A signature like `def findUser(id: Long): Option[User]` looks precise, but `Long` carries no domain meaning — an `OrderId`, a `Long` read from a config file, and a raw row count are all the exact same type as far as the compiler is concerned. Nothing stops a caller from passing an `orderId` where a `userId` is expected; both type-check identically, and the mistake only surfaces once the wrong record comes back (or doesn't) at runtime. This is primitive obsession: real domain concepts — an id, an email, a currency amount — represented as whatever general-purpose primitive happens to hold their value, with the distinction between them living only in a parameter name a caller can ignore.

Scala 3's `opaque type` fixes this with no runtime cost: `opaque type UserId = Long` declares `UserId` as a genuinely distinct type everywhere outside the scope where it's defined, while being literally a `Long` at runtime — no wrapper object, no boxing, no allocation. Two opaque types with the same underlying representation (`UserId` and `OrderId`, both backed by `Long`) are still mutually incompatible to the compiler, so passing one where the other is expected is a compile error, not a bug waiting to be found. A companion `apply` and an `extension` method are the idiomatic way to construct and unwrap the value at the boundary where the underlying representation is actually needed (serialization, a database driver, a log line), while ordinary domain code passes the opaque type itself around without ever touching the primitive underneath.

## Bad Example
```scala
def findUser(id: Long): Option[User] = users.get(id)
def findOrder(id: Long): Option[Order] = orders.get(id)

findUser(order.id) // compiles: both are plain Long, nothing distinguishes a user id from an order id
```

## Good Example
```scala
opaque type UserId = Long
object UserId:
  def apply(raw: Long): UserId = raw
  extension (id: UserId) def raw: Long = id

opaque type OrderId = Long
object OrderId:
  def apply(raw: Long): OrderId = raw
  extension (id: OrderId) def raw: Long = id

def findUser(id: UserId): Option[User] = users.get(id.raw)

findUser(OrderId(42)) // does not compile: OrderId is not a UserId, even though both wrap Long
```

## Notes
- Opaque types are transparent (equal to their underlying type) only within the scope where they're defined — outside that scope they're fully abstract, which is what forces callers through the companion's constructor and extension methods.
- This is unrelated to validating the wrapped value — an opaque type only buys nominal distinctness; pair it with `api-design-smart-constructors` when the primitive also needs an invariant enforced (a well-formed email, a positive amount).
- Prefer this over a `case class UserId(value: Long) extends AnyVal`-style value class for a single-field wrapper: an opaque type never needs `AnyVal`'s single-constructor-parameter restrictions and never risks the boxing that value classes fall back to in generic contexts.
- Keep the opaque type's defining `object` (`UserId` above) as the one place that ever sees the raw representation; everywhere else should accept and return `UserId`.

## References
- [Scala 3 Reference — Opaque Types](https://docs.scala-lang.org/scala3/reference/other-new-types/opaques.html)
