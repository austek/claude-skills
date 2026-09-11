---
title: Structure Test Bodies as Arrange-Act-Assert
impact: MEDIUM
impactDescription: cuts time-to-diagnose on a failing test by keeping setup, action, and verification visually separate
tags: [scalatest, munit, structure, readability]
---

# Structure Test Bodies as Arrange-Act-Assert [MEDIUM]

## Description
A test body reads fastest when it has exactly three visible sections, in order: build the inputs and collaborators (Arrange), invoke the one thing under test (Act), check the outcome (Assert). Separate the sections with a blank line and keep each one to what it needs — no assertion mixed into setup, no further setup after the act. This convention is orthogonal to the test framework: it applies the same way inside a MUnit `test("...")` block and a ScalaTest `AnyFunSuite` or `AnyWordSpec` test body.

The payoff shows up when a test fails. A reader scanning top to bottom can tell at a glance which line built the fixture, which line exercised the behavior, and which line's expectation didn't hold, without tracing variable mutations across the whole body. Interleaving these — asserting partway through setup, or building more state after the act — forces the reader to re-derive that structure from scratch on every failure.

## Bad Example
```scala
test("checkout applies a discount") {
  val cart = Cart.empty.addItem(Item("sku-1", price = 10.0))
  assert(cart.items.size == 1)
  val discounted = Checkout.applyDiscount(cart, DiscountCode("SAVE10"))
  val total = discounted.total
  assert(total == 9.0)
  assert(discounted.appliedCode.contains(DiscountCode("SAVE10")))
}
```

## Good Example
```scala
test("checkout applies a discount") {
  val cart = Cart.empty.addItem(Item("sku-1", price = 10.0))

  val discounted = Checkout.applyDiscount(cart, DiscountCode("SAVE10"))

  assertEquals(discounted.total, 9.0)
  assertEquals(discounted.appliedCode, Some(DiscountCode("SAVE10")))
}
```

## Notes
- A blank line between sections is enough signal — no `// Arrange` / `// Act` / `// Assert` comments needed once the pattern is habitual.
- Multiple `assert`/`assertEquals` calls at the end are fine; they're all still the Assert section as long as none of them trigger new setup.
- `scalatest-munit-structure-one-behavior-per-test` keeps the Act step to a single call, which is what makes this structure possible in the first place.

## References
- [MUnit — Basics](https://scalameta.org/munit/docs/basics.html)
- [ScalaTest — FunSuite](https://www.scalatest.org/getting_started_with_fun_suite)
