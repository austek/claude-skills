---
title: Provide Arbitrary Instances for Domain Types
impact: MEDIUM
impactDescription: lets forAll infer generators for domain types automatically instead of threading explicit Gen values through every property
tags: [scalacheck, arbitrary, domain-modeling]
---

# Provide Arbitrary Instances for Domain Types [MEDIUM]

## Description
For a domain type used across several properties (`OrderId`, `Money`, `EmailAddress`), define a `given Arbitrary[A]` once, next to the type or in a shared test-support module, instead of writing an explicit `Gen[A]` and passing it into every `forAll` that needs one. `forAll { (a: A) => ... }` resolves its generator implicitly from `Arbitrary[A]` — with an instance in scope, every property that mentions the type gets a correctly-shaped generator for free, the same way a `given Ordering[A]` makes every `.sorted` call on `List[A]` work without an explicit argument.

Keep the `Arbitrary` instance itself built from `scalacheck-generator-composition`-style composed generators so it only ever produces valid values of the type — an `Arbitrary[Order]` should never generate an `Order` with a negative total if the constructor doesn't allow one. Where a type has more than one plausible generation strategy for different test scenarios (e.g. "any customer" vs. "a customer with no orders"), keep the general case as the `given Arbitrary` default and expose the narrower cases as named `Gen` values instead of trying to encode all of them into one `Arbitrary`.

## Bad Example
```scala
def genOrderId: Gen[OrderId] = Gen.identifier.map(OrderId.apply)
def genCustomer: Gen[Customer] = /* ... */

property("order ids are stable across serialization") {
  forAll(genOrderId) { id => Codec.decode(Codec.encode(id)) == id }
}
property("customer ids are stable across serialization") {
  forAll(genCustomer) { customer => Codec.decode(Codec.encode(customer)) == customer }
}
// every property call site must remember and thread the right Gen explicitly
```

## Good Example
```scala
given Arbitrary[OrderId] = Arbitrary(Gen.identifier.map(OrderId.apply))
given Arbitrary[Customer] = Arbitrary(for
  id    <- Gen.identifier
  email <- genEmail
yield Customer(id, email))

property("order ids are stable across serialization") {
  forAll { (id: OrderId) => Codec.decode(Codec.encode(id)) == id }
}
property("customers are stable across serialization") {
  forAll { (customer: Customer) => Codec.decode(Codec.encode(customer)) == customer }
}
```

## Notes
- Define domain `Arbitrary` instances in a shared test-support object imported by every suite that needs them, not duplicated per-suite.
- An `Arbitrary[A]` that can produce an invalid `A` (one the smart constructor would reject) makes every property using it test the wrong thing; build it from validated construction, not `A.apply` directly on raw fields.
- [`api-design-smart-constructors`](../../scala-coding-standards/rules/api-design-smart-constructors.md) covers keeping construction validated in the first place, which is what a well-formed `Arbitrary` instance should rely on.

## References
- [ScalaCheck — User Guide, Arbitrary](https://github.com/typelevel/scalacheck/blob/main/doc/UserGuide.md#generating-arbitrary-values)
