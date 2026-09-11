---
title: Match ScalaTest Fixture Traits to Setup Cost
impact: MEDIUM
impactDescription: avoids paying per-test setup cost for a resource that's actually safe to share for a whole suite
tags: [scalatest, fixtures, lifecycle]
---

# Match ScalaTest Fixture Traits to Setup Cost [MEDIUM]

## Description
ScalaTest offers `BeforeAndAfterEach` (runs `beforeEach`/`afterEach` around every single test) and `BeforeAndAfterAll` (runs `beforeAll`/`afterAll` once for the whole suite). Pick based on what the resource actually costs and whether it's safe to share: cheap, test-owned state (a fresh in-memory collection, a mutable counter reset to zero) belongs in `BeforeAndAfterEach`, so no test can observe another test's leftovers. Expensive, read-only, or externally-managed state (a started test container, a compiled schema) belongs in `BeforeAndAfterAll`, since re-creating it per test would dominate the suite's runtime for no correctness benefit.

The trap is mixing the two incorrectly: putting mutable state that tests *write to* behind `BeforeAndAfterAll` makes tests order-dependent, because nothing resets it between them. If a `BeforeAndAfterAll`-scoped resource must be written to by tests, make sure each test resets the part it touched, or restrict that resource to read-only use and keep genuinely per-test mutable state in `BeforeAndAfterEach`.

## Bad Example
```scala
class OrderServiceSuite extends AnyFunSuite with BeforeAndAfterAll:
  private var repo: InMemoryOrderRepository = _

  override def beforeAll(): Unit = repo = InMemoryOrderRepository.empty

  test("saves an order") {
    repo.save(sampleOrder)
    assert(repo.findById(sampleOrder.id).contains(sampleOrder))
  }

  test("starts with no orders") {
    // fails or passes depending on whether the previous test already ran
    assert(repo.findAll().isEmpty)
  }
```

## Good Example
```scala
class OrderServiceSuite extends AnyFunSuite with BeforeAndAfterEach:
  private var repo: InMemoryOrderRepository = _

  override def beforeEach(): Unit = repo = InMemoryOrderRepository.empty

  test("saves an order") {
    repo.save(sampleOrder)
    assert(repo.findById(sampleOrder.id).contains(sampleOrder))
  }

  test("starts with no orders") {
    assert(repo.findAll().isEmpty)
  }
```

## Notes
- The loan-fixture pattern (`withFixture(OneArgTest)`) is a further option when a resource needs guaranteed cleanup even if `beforeEach` itself throws — `BeforeAndAfterEach` alone doesn't protect against setup failing partway through.
- Call `super.beforeEach()`/`super.afterEach()` when stacking more than one fixture trait, or the later trait's hooks silently never run.
- `fixtures-munit-lifecycle` covers the equivalent trade-off in MUnit, where `FunFixture` is generally preferred to either ScalaTest hook style.

## References
- [ScalaTest — BeforeAndAfterEach](https://www.scalatest.org/scaladoc/3.2.17/org/scalatest/BeforeAndAfterEach.html)
- [ScalaTest — Sharing fixtures](https://www.scalatest.org/user_guide/sharing_fixtures)
