---
title: Let CatsEffectSuite Run IO Tests, Don't Call unsafeRunSync Yourself
impact: HIGH
impactDescription: keeps test cancellation, timeouts, and error reporting under the test runner's control instead of a hand-rolled run call
tags: [async-testing, cats-effect, munit]
---

# Let CatsEffectSuite Run IO Tests, Don't Call unsafeRunSync Yourself [HIGH]

## Description
`munit.CatsEffectSuite` (from `munit-cats-effect`) lets a test body return `IO[Unit]` directly — MUnit detects the effect via `munitValueTransform` and awaits it as part of running the test, failing the test if the `IO` raises an error and applying the suite's configured test timeout to it. Extend `CatsEffectSuite` and write `test("name") { myProgram.map(result => assertEquals(result, expected)) }`, not `test("name") { myProgram.unsafeRunSync(); ... }`.

Calling `.unsafeRunSync()` manually inside a plain `munit.FunSuite` test works for the common case, but it opts out of everything the runner provides for free: the suite's configured munit timeout no longer applies (a hang blocks the whole suite instead of failing one test), and an asynchronous failure inside nested fibers can surface as a confusing wrapped exception instead of the runner's normal MUnit failure report. `CatsEffectSuite` exists specifically so IO-returning code is tested the same way synchronous code is — return the value, let the runner do the awaiting.

## Bad Example
```scala
class OrderServiceSuite extends munit.FunSuite:
  test("confirms an order") {
    val result = OrderService(repo).confirm(orderId).unsafeRunSync()
    assertEquals(result.status, OrderStatus.Confirmed)
    // no suite-level timeout applies to this call; a hang here blocks the whole run
  }
```

## Good Example
```scala
class OrderServiceSuite extends munit.CatsEffectSuite:
  test("confirms an order") {
    OrderService(repo).confirm(orderId).map { result =>
      assertEquals(result.status, OrderStatus.Confirmed)
    }
  }
```

## Notes
- `CatsEffectSuite` also accepts a test body returning a plain `Unit` for synchronous tests in the same suite — it isn't all-or-nothing per suite.
- Override `munitTimeout` on the suite to change the per-test timeout `CatsEffectSuite` enforces, rather than reaching for a manual timeout wrapper.
- `async-testing-deterministic-clocks` covers testing time-dependent `IO` programs without waiting on real wall-clock time inside these tests.

## References
- [munit-cats-effect](https://typelevel.org/munit-cats-effect/)
