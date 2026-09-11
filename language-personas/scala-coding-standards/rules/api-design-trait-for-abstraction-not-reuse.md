---
title: Use a Trait to Declare an Abstraction, Not to Share an Implementation
impact: MEDIUM
impactDescription: keeps a type's inheritance list meaningful as "what this can substitute for" instead of "what it happened to borrow code from"
tags: [api-design, traits, inheritance, composition]
---

# Use a Trait to Declare an Abstraction, Not to Share an Implementation [MEDIUM]

## Description
A trait can do two very different jobs, and conflating them produces fragile hierarchies. The first job is declaring an abstraction: a contract — `PaymentGateway`, `UserRepository` — that several unrelated concrete types can implement, each substitutable for the others wherever the trait type is expected. The second job, which a trait can also technically do because it can carry concrete method bodies, is bundling a piece of reusable implementation (`trait Loggable { def log(msg: String) = println(msg) }`) and mixing it into classes purely so they inherit that method. The second use reaches for inheritance to solve what is really a code-sharing problem, and it comes with everything inheritance costs: the mixing class's public surface grows with methods it didn't design, `super` calls become ambiguous once two such traits are mixed together, and nothing about "this class extends `Loggable`" tells a reader whether `Loggable` is part of the class's actual contract or just where one method's body happens to live.

Prefer a plain method call — a companion object function, a passed-in collaborator — for sharing implementation, and reserve traits for the abstraction role: a contract with (ideally) more than one real implementation, where substitutability is the actual point. `implicits-givens-extension-methods-over-implicit-class` covers the parallel case of adding a method to an *existing* type without inheritance; the same instinct — prefer composition to a mixin — applies when designing a brand-new type's own dependencies.

## Bad Example
```scala
trait Loggable:
  def log(msg: String): Unit = println(msg) // pulled in for its body, not as a substitutable abstraction

class OrderService extends Loggable:
  def place(order: Order): Unit = log(s"placing ${order.id}")

class PaymentService extends Loggable:
  def charge(amount: BigDecimal): Unit = log(s"charging $amount")
```

## Good Example
```scala
trait PaymentGateway: // a genuine abstraction: multiple real implementations substitute for it
  def charge(amount: BigDecimal): Either[PaymentError, Receipt]

object ConsoleLogger:
  def log(msg: String): Unit = println(msg) // reuse via a plain call, not inheritance

class PaymentService(gateway: PaymentGateway):
  def charge(amount: BigDecimal): Either[PaymentError, Receipt] =
    ConsoleLogger.log(s"charging $amount")
    gateway.charge(amount)
```

## Notes
- A quick test: if a trait has ever had, or could plausibly have, more than one concrete implementation substituted at a call site, it's an abstraction. If it exists only so classes can `extends` their way to one shared method body, it's reuse wearing an abstraction's syntax.
- Traits with no concrete members at all (pure interfaces) are always the abstraction case — this rule only concerns traits that carry implementation.
- `implicits-givens-type-class-pattern` covers a different way to add capability to a type — via a `given` instance rather than inheritance — which sidesteps this problem entirely for capabilities that don't need per-instance state.
- Composition costs one extra constructor parameter; inheritance costs a permanent, hard-to-unwind coupling between two classes' hierarchies — the trade is rarely close.

## References
- [Scala 3 Book — Traits](https://docs.scala-lang.org/scala3/book/domain-modeling-tools.html#traits)
