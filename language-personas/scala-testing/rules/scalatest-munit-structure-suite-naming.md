---
title: Name Test Suites After the Unit Under Test
impact: LOW
impactDescription: lets a failing suite name alone point to the right source file
tags: [scalatest, munit, structure, naming]
---

# Name Test Suites After the Unit Under Test [LOW]

## Description
Name a test suite class `<UnitUnderTest>Spec` or `<UnitUnderTest>Suite` — pick one suffix per codebase and stay consistent — where `<UnitUnderTest>` is the exact production class or object being tested. `OrderService` gets `OrderServiceSpec`; a pure function module `PricingRules` gets `PricingRulesSpec`. Avoid generic names (`ServiceTest`, `Tests`, `Checks`) and avoid bundling unrelated units into one suite for convenience — one suite per unit keeps the mapping obvious in CI output, IDE test trees, and `sbt testOnly` invocations.

The suffix also documents which framework style is in play without opening the file: `Spec` conventionally signals behavior-style suites (`AnyWordSpec`, `AnyFlatSpec`) while `Suite` fits `AnyFunSuite` and MUnit's `FunSuite`, though this is a house convention, not something either framework enforces. What matters more than the exact suffix is that a developer reading a stack trace like `OrderServiceSpec` can immediately guess `OrderService.scala` is the file to open, and that `sbt "testOnly *OrderService*"` reliably matches both.

## Bad Example
```scala
class Tests extends munit.FunSuite:
  test("order total includes tax") { /* ... */ }
  test("user email must be valid") { /* ... */ }
```

## Good Example
```scala
class OrderServiceSuite extends munit.FunSuite:
  test("order total includes tax") { /* ... */ }

class UserValidationSuite extends munit.FunSuite:
  test("user email must be valid") { /* ... */ }
```

## Notes
- Keep the suffix uniform across a codebase; a mix of `Spec`/`Suite`/`Test` on otherwise-identical suites makes glob-based test selection unreliable.
- A suite testing a trait's default methods can name itself after the trait, or after a concrete test double implementing it (`InMemoryOrderRepositorySpec`) when the trait itself has no behavior to assert on.
- `scalatest-munit-structure-nested-suites` still applies the same per-unit naming to each nested suite it composes.

## References
- [ScalaTest — Suite naming conventions](https://www.scalatest.org/user_guide/selecting_a_style)
