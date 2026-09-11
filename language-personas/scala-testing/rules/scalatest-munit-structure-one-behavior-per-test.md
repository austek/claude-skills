---
title: Assert One Behavior per Test
impact: HIGH
impactDescription: a failing test name alone identifies the broken behavior, no need to read assertions to localize the bug
tags: [scalatest, munit, structure, isolation]
---

# Assert One Behavior per Test [HIGH]

## Description
Give each `test(...)` block a single reason to fail: one action, one observable behavior, described by the test name. A test named `"rejects an order when the cart is empty"` should exercise exactly that path and assert only what that sentence promises — not also check tax calculation or shipping cost in the same block because a `Cart` happened to be lying around. When a test covers several behaviors, a single assertion failure hides whichever other behaviors it never got to check, and a single broken behavior can fail several unrelated-looking tests at once, making the test report a worse map of what's actually wrong than the code itself.

This is not a ban on multiple `assert`/`assertEquals` calls — asserting several facets of one outcome (`discounted.total`, `discounted.appliedCode`) is still one behavior. The line to hold is: does this test need more than one call to the thing under test, or more than one English sentence, to describe what it checks? If so, split it. Splitting also makes each test's fixture smaller, since it only needs to set up the state relevant to its one behavior.

## Bad Example
```scala
test("order processing") {
  val order = Order(items = List(Item("sku-1", 10.0)), customer = verifiedCustomer)
  val processed = OrderService.process(order)
  assertEquals(processed.total, 10.0)
  assertEquals(processed.status, OrderStatus.Confirmed)

  val emptyOrder = Order(items = Nil, customer = verifiedCustomer)
  assertEquals(OrderService.process(emptyOrder).status, OrderStatus.Rejected)
}
```

## Good Example
```scala
test("confirms an order with a non-empty cart") {
  val order = Order(items = List(Item("sku-1", 10.0)), customer = verifiedCustomer)

  val processed = OrderService.process(order)

  assertEquals(processed.status, OrderStatus.Confirmed)
}

test("rejects an order with an empty cart") {
  val order = Order(items = Nil, customer = verifiedCustomer)

  val processed = OrderService.process(order)

  assertEquals(processed.status, OrderStatus.Rejected)
}
```

## Notes
- A test name that needs "and" to describe it (`"confirms and prices the order"`) is a signal to split into two tests.
- This rule composes with `scalatest-munit-structure-aaa-pattern`: a single-behavior test is what makes a clean Arrange-Act-Assert body possible.
- Shared setup across several single-behavior tests belongs in a fixture (`fixtures-munit-lifecycle`, `fixtures-scalatest-beforeandafter-traits`), not in a bigger test.

## References
- [MUnit — Assertions](https://scalameta.org/munit/docs/assertions.html)
