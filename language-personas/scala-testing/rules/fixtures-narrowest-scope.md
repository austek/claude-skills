---
title: Scope Fixtures to the Narrowest Level That Needs Them
impact: MEDIUM
impactDescription: eliminates order-dependent test failures caused by state that outlives the test that needs it
tags: [fixtures, scope, isolation]
---

# Scope Fixtures to the Narrowest Level That Needs Them [MEDIUM]

## Description
Create a fixture at the smallest scope that satisfies its purpose — per-test by default, per-suite only when the resource is expensive to build and genuinely read-only or reset between uses, and global (shared across suites) only for something like a one-time compiled schema or a test-container that's costly enough to amortize across an entire run. Widening scope "to save setup time" is a trade against correctness: the wider a fixture's scope, the more tests implicitly depend on the order they run in and on each other's cleanup discipline, and the harder a failure is to reproduce in isolation.

Default to per-test (`fixtures-munit-lifecycle`'s `FunFixture`, or ScalaTest's `BeforeAndAfterEach`) and only widen the scope when profiling actually shows setup cost dominating the suite's runtime — and even then, prefer making the per-suite resource immutable/read-only over letting tests mutate a shared instance, since a read-only shared fixture keeps the correctness properties of a narrow one.

## Bad Example
```scala
object SharedFixtures:
  val repo: InMemoryOrderRepository = InMemoryOrderRepository.empty // module-level, shared across every suite in the run

class OrderServiceSuite extends munit.FunSuite:
  test("saves an order") {
    SharedFixtures.repo.save(sampleOrder)
    assert(SharedFixtures.repo.findById(sampleOrder.id).isDefined)
  }
  // any other suite's test that also touches SharedFixtures.repo can observe this order
```

## Good Example
```scala
class OrderServiceSuite extends munit.FunSuite:
  private val repoFixture = FunFixture[InMemoryOrderRepository](
    setup = _ => InMemoryOrderRepository.empty,
    teardown = _ => ()
  )

  repoFixture.test("saves an order") { repo =>
    repo.save(sampleOrder)
    assert(repo.findById(sampleOrder.id).isDefined)
  }
```

## Notes
- A resource that's expensive to start but never mutated by tests (a Postgres test container, queried but not written to) is a legitimate candidate for suite- or run-level scope.
- If a suite-scoped resource must be written to, reset the specific state each test touched in that test's own teardown rather than assuming the next test won't notice.
- An in-memory test double is usually cheap enough to build fresh per test that there's rarely a performance reason to widen its scope beyond that.

## References
- [xUnit Test Patterns — Fixture Setup Patterns](http://xunitpatterns.com/fixture%20setup%20patterns.html)
