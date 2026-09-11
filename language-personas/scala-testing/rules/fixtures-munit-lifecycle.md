---
title: Prefer FunFixture Over Mutable beforeEach/afterEach in MUnit
impact: MEDIUM
impactDescription: removes the shared mutable var that makes test order and parallel execution unsafe
tags: [munit, fixtures, lifecycle]
---

# Prefer FunFixture Over Mutable beforeEach/afterEach in MUnit [MEDIUM]

## Description
MUnit supports two ways to give tests a fresh resource: overriding `beforeEach`/`afterEach` and stashing the resource in a `var` field, or declaring a `FunFixture[A]` with `setup`/`teardown` functions and calling `.test(name) { resource => ... }` instead of the plain `test`. Prefer `FunFixture`. It threads the resource into the test body as a parameter, so there's no mutable field for tests to race on under MUnit's parallel execution, and no way for one test to observe a resource left over by a previous test's incomplete teardown — each `.test` call gets its own `setup`/`teardown` pair, scoped exactly to that one test.

The `var`-based `beforeEach` approach also silently couples every test in the suite to remembering to reset every field it touches; forgetting one produces a test that only fails depending on execution order — one of the hardest categories of test bug to reproduce locally. `FunFixture` makes that class of bug structurally impossible, since there's no field left over to forget.

## Bad Example
```scala
class OrderRepositorySuite extends munit.FunSuite:
  private var repo: InMemoryOrderRepository = _

  override def beforeEach(context: BeforeEach): Unit =
    repo = InMemoryOrderRepository.empty

  test("saves and retrieves an order") {
    repo.save(sampleOrder)
    assertEquals(repo.findById(sampleOrder.id), Some(sampleOrder))
  }
```

## Good Example
```scala
class OrderRepositorySuite extends munit.FunSuite:
  private val repoFixture = FunFixture[InMemoryOrderRepository](
    setup = _ => InMemoryOrderRepository.empty,
    teardown = _ => ()
  )

  repoFixture.test("saves and retrieves an order") { repo =>
    repo.save(sampleOrder)
    assertEquals(repo.findById(sampleOrder.id), Some(sampleOrder))
  }
```

## Notes
- `FunFixture.map`/`FunFixture.andThen` compose two fixtures (e.g. a temp directory plus a database connection) without any manual field bookkeeping.
- `beforeEach`/`afterEach` still have a place for genuinely suite-wide, stateless setup (configuring a logger level) that no test reads back as data.
- `fixtures-narrowest-scope` covers choosing between a per-test `FunFixture` and a per-suite resource in the first place.

## References
- [MUnit — Fixtures](https://scalameta.org/munit/docs/fixtures.html)
