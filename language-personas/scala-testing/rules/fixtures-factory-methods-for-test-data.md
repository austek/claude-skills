---
title: Build Test Data with Factory Methods, Not Copy-Pasted Literals
impact: HIGH
impactDescription: one call site to update when a case class gains a field, instead of every test that constructs it
tags: [fixtures, test-data, factory-methods]
---

# Build Test Data with Factory Methods, Not Copy-Pasted Literals [HIGH]

## Description
Give every domain type used across more than a couple of tests a factory method with sensible defaults for every field — `def sampleOrder(items: List[Item] = List(defaultItem), customer: Customer = sampleCustomer): Order` — and have tests call it, overriding only the field their scenario actually cares about. This does two things a literal `Order(List(Item("sku-1", 10.0)), Customer("c1", "a@b.com"))` repeated in fifty tests can't: it makes the field under test visible at each call site (`sampleOrder(items = Nil)` reads as "an order with no items", the point of the test), and it means a new required field on `Order` is a one-line change to the factory instead of fifty broken tests.

Keep factories composable the same way constructors are: a factory for a type that contains another domain type should default that field via the nested type's own factory (`sampleOrder`'s `customer` default calls `sampleCustomer()`), not duplicate its construction. This is the test-code analogue of [`api-design-companion-object-factories`](../../scala-coding-standards/rules/api-design-companion-object-factories.md) in production code — a single, named, default-supplying entry point instead of scattered literal construction.

## Bad Example
```scala
test("rejects an order for an unverified customer") {
  val order = Order(List(Item("sku-1", 10.0)), Customer("c1", "a@b.com", verified = false))
  assertEquals(OrderService.process(order).status, OrderStatus.Rejected)
}

test("confirms an order for a verified customer") {
  val order = Order(List(Item("sku-1", 10.0)), Customer("c1", "a@b.com", verified = true))
  assertEquals(OrderService.process(order).status, OrderStatus.Confirmed)
}
// every field is repeated at every call site, and only `verified` is actually relevant to either test
```

## Good Example
```scala
def sampleCustomer(verified: Boolean = true): Customer =
  Customer(id = "c1", email = "a@b.com", verified = verified)

def sampleOrder(items: List[Item] = List(Item("sku-1", 10.0)), customer: Customer = sampleCustomer()): Order =
  Order(items, customer)

test("rejects an order for an unverified customer") {
  val order = sampleOrder(customer = sampleCustomer(verified = false))
  assertEquals(OrderService.process(order).status, OrderStatus.Rejected)
}

test("confirms an order for a verified customer") {
  val order = sampleOrder()
  assertEquals(OrderService.process(order).status, OrderStatus.Confirmed)
}
```

## Notes
- Place shared factories in a `trait` or `object` that suites mix in or import, so the same defaults are used across the whole test tree, not redefined per file.
- Factory defaults should represent the "boring", valid, happy-path case — a scenario overrides only what makes it different.
- `scalacheck-arbitrary-instances-for-domain-types` covers the property-based equivalent of this rule, where the "factory" is a generator instead of a fixed default.

## References
- [xUnit Test Patterns — Test Data Builder](http://xunitpatterns.com/Test%20Data%20Builder.html)
