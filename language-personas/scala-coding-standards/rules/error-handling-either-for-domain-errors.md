---
title: Model Domain Errors with Either[E, A]
impact: HIGH
impactDescription: makes every expected failure mode part of the compiler-checked return type
tags: [error-handling, either, domain-modeling]
---

# Model Domain Errors with Either[E, A] [HIGH]

## Description
A business operation — placing an order, validating a registration, authorizing a transition — usually has more than one way to fail, and each failure is something the calling code is expected to handle differently: show a specific message, decline the request, retry, log and move on. Model that directly in the return type as `Either[E, A]`, where `E` is a small, closed error type describing every domain failure the operation can produce (`error-handling-typed-errors-over-strings`) and `A` is the success value. This puts the full set of expected outcomes — one success shape, N failure shapes — in the signature itself, so nothing about handling failure depends on documentation or tribal knowledge of what the method "can" throw.

Multi-step domain operations compose the same way single validations do: chain `Either`-returning steps in a `for`-comprehension, and the computation short-circuits on the first `Left` without any manual propagation code. This gives the same shape as exception-based control flow (stop at the first failure, skip the rest) but with the failure type visible and checked at every step, and without unwinding a stack or paying exception-construction cost for what is, in domain terms, a completely ordinary outcome. Reserve throwing (or an effect type's error channel, `error-handling-io-for-effectful-errors`) for failures that are not part of the domain's normal vocabulary — a database connection dropping mid-transaction is not the same kind of failure as a customer's cart being empty.

## Bad Example
```scala
class OrderService:
  def placeOrder(cart: Cart, customer: Customer): Order =
    if cart.items.isEmpty then throw new IllegalStateException("cart is empty")
    if !customer.isVerified then throw new IllegalStateException("customer not verified")
    Order(cart, customer)
// caller must know, from reading the implementation, which exceptions to catch and why
```

## Good Example
```scala
enum OrderError:
  case EmptyCart
  case UnverifiedCustomer(customerId: String)

class OrderService:
  def placeOrder(cart: Cart, customer: Customer): Either[OrderError, Order] =
    for
      _ <- Either.cond(cart.items.nonEmpty, (), OrderError.EmptyCart)
      _ <- Either.cond(customer.isVerified, (), OrderError.UnverifiedCustomer(customer.id))
    yield Order(cart, customer)

orderService.placeOrder(cart, customer).fold(
  {
    case OrderError.EmptyCart               => respondBadRequest("cart is empty")
    case OrderError.UnverifiedCustomer(id)  => respondForbidden(s"customer $id not verified")
  },
  order => respondCreated(order)
)
```

## Notes
- `Either.cond(test, right, left)` is a concise way to express one validation step without an explicit `if`/`else` producing `Left`/`Right`.
- Keep `Either` for *expected* domain failures the caller is meant to branch on programmatically — not for programmer bugs, missing invariants, or infrastructure failures, which belong to `error-handling-no-throw-for-control-flow`'s narrower exception carve-out or to an effect type's error channel.
- A service layer method returning `Either[DomainError, A]` should not also throw checked or unchecked exceptions for the same category of failure — pick one signal per failure category and stay consistent across the method.
- `null-option-either-for-expected-failure` covers the same `Either` mechanics at the level of a single operation; this rule is about using it as the standard shape for a domain service's public API.

## References
- [Scala 3 Standard Library — Either](https://www.scala-lang.org/api/current/scala/util/Either.html)
