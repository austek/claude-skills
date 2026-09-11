---
title: Nest Suites by Shared Context, Not by File Convenience
impact: MEDIUM
impactDescription: keeps a failing nested test's printed description a complete sentence describing the exact scenario
tags: [scalatest, munit, structure, nesting]
---

# Nest Suites by Shared Context, Not by File Convenience [MEDIUM]

## Description
`AnyWordSpec`'s nested `should`/`when`/`in` blocks and MUnit's plain flat `test(...)` calls both describe the same suite structure differently — the choice to nest should track a genuinely shared scenario (a fixed starting state, a specific precondition), not just a desire to group tests visually. A `AnyWordSpec` block like `"an empty cart" when { "checked out" should { "be rejected" in { ... } } }` is doing real work: it builds a sentence describing the exact circumstance under test, and it lets sibling blocks reuse the outer `"an empty cart"` context. Nesting purely for indentation, with no shared setup or shared precondition between siblings, adds ceremony without adding information.

MUnit has no nesting construct by design — flat `test("an empty cart is rejected at checkout")` names carry the same information in the string itself. Choosing `AnyWordSpec` over `AnyFunSuite`/MUnit is a decision about whether the shared-context structure earns its extra nesting, not a style preference to apply uniformly across a codebase.

## Bad Example
```scala
"OrderService" should {
  "processing an order" should {
    "with a valid cart" should {
      "confirm it" in {
        OrderService.process(validOrder).status shouldBe OrderStatus.Confirmed
      }
    }
  }
}
// three levels of nesting for one test, none of them shared with a sibling
```

## Good Example
```scala
"an empty cart" when {
  "checked out" should {
    "be rejected" in {
      OrderService.process(emptyOrderCart).status shouldBe OrderStatus.Rejected
    }
    "not charge the customer" in {
      OrderService.process(emptyOrderCart).charge shouldBe None
    }
  }
}
// "an empty cart" and "checked out" are genuinely shared by both assertions
```

## Notes
- If no level of nesting has more than one child, flatten it — a single-child nest is pure indentation.
- MUnit suites that want shared context between tests should reach for a fixture (`fixtures-munit-lifecycle`) rather than trying to simulate nesting.
- `scalatest-munit-structure-suite-naming` still applies per top-level suite; nesting is a within-suite concern.

## References
- [ScalaTest — WordSpec](https://www.scalatest.org/getting_started_with_word_spec)
