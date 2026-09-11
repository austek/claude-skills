---
title: Compose Generators from Smaller Generators
impact: MEDIUM
impactDescription: keeps generated test data valid by construction instead of filtered after the fact
tags: [scalacheck, generators, composition]
---

# Compose Generators from Smaller Generators [MEDIUM]

## Description
Build a `Gen[DomainType]` for a case class or ADT by mapping and flat-mapping smaller generators for its fields, the same way `for`-comprehensions compose `Option` or `Either` — never by generating a loosely-typed value and validating/filtering it afterward. `Gen` has `map`, `flatMap`, and `for`-comprehension sugar precisely so that a `Gen[Order]` can be assembled from a `Gen[List[Item]]`, a `Gen[Customer]`, and a `Gen[DiscountCode]` that already exist for testing those types individually, each generator staying responsible only for its own field's valid range.

Composition also keeps generators shrink-friendly (`scalacheck-shrinking-friendly-generators`) and keeps the distribution sane: `Gen.filter` (`Gen.suchThat`) discards generated values that fail the predicate and can exhaust ScalaCheck's discard budget if the filter is narrow, whereas a generator built from correctly-bounded sub-generators never produces an invalid value in the first place. Reach for `Gen.oneOf`, `Gen.choose`, and `Gen.listOfN` to bound each field's own generator, then compose the results — don't generate broadly and filter down.

## Bad Example
```scala
val genOrder: Gen[Order] =
  (for
    items <- Gen.listOf(Gen.const(Item("sku-1", 10.0)))
    email <- Gen.alphaStr
  yield Order(items, Customer(email)))
    .suchThat(_.customer.email.nonEmpty) // filters the whole Gen[Order] after the fact instead of bounding Customer's own generator
```

## Good Example
```scala
val genItem: Gen[Item] =
  for
    sku   <- Gen.identifier
    price <- Gen.choose(0.01, 999.99)
  yield Item(sku, price)

val genEmail: Gen[String] =
  for
    user   <- Gen.identifier
    domain <- Gen.oneOf("example.com", "test.org")
  yield s"$user@$domain"

val genCustomer: Gen[Customer] = genEmail.map(Customer.apply)

val genOrder: Gen[Order] =
  for
    items    <- Gen.listOf(genItem)
    customer <- genCustomer
  yield Order(items, customer)
```

## Notes
- `Gen.listOfN(n, gen)` and `Gen.containerOf[List, A](gen)` give control over collection size where `Gen.listOf`'s default distribution is too broad.
- Reserve `Gen.suchThat`/`retryUntil` for constraints that genuinely can't be expressed generatively (e.g. "two distinct elements from a small enum"), not as a substitute for a tighter sub-generator.
- A named `genX: Gen[X]` value per domain type is reusable across every property that needs an `X`, the same way a factory method is reused across example-based tests (`fixtures-factory-methods-for-test-data`).

## References
- [ScalaCheck — User Guide, Generators](https://github.com/typelevel/scalacheck/blob/main/doc/UserGuide.md#generators)
