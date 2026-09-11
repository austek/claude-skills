---
title: Tag Slow or Environment-Dependent Tests
impact: MEDIUM
impactDescription: lets a fast local run exclude minutes-long or infra-dependent tests without deleting them
tags: [scalatest, munit, structure, tags]
---

# Tag Slow or Environment-Dependent Tests [MEDIUM]

## Description
Give tests that are slow, flaky-by-nature, or dependent on external infrastructure (a real database, network access) an explicit tag, and let the build decide when to run them, rather than commenting them out or leaving them mixed in with the fast unit suite. Both frameworks support this natively: ScalaTest's `Tag` combined with `taggedAs`/`test(name, tags*)`, and MUnit's `Tag` combined with `.tag(...)` on the test name. A default local or pre-commit run excludes the tag; CI (or an explicit `testOnly` invocation) includes it.

Tagging is preferable to a separate `integrationTest` source set for cases that don't warrant a whole new compilation unit — a handful of tests that happen to need a running container, say. It's also preferable to silently letting slow tests degrade the fast feedback loop that unit tests exist to provide: a suite of 500 fast tests plus 5 minute-long integration tests trains developers to stop running tests locally at all.

## Bad Example
```scala
// no tag: this always runs with the rest of the suite, including on every local `sbt test`
test("repository round-trips a record through the real Postgres instance") {
  val repo = PostgresOrderRepository(testDataSource)
  repo.save(sampleOrder)
  assertEquals(repo.findById(sampleOrder.id), Some(sampleOrder))
}
```

## Good Example
```scala
import munit.Tag

object Slow extends Tag("slow")

test("repository round-trips a record through the real Postgres instance".tag(Slow)) {
  val repo = PostgresOrderRepository(testDataSource)
  repo.save(sampleOrder)
  assertEquals(repo.findById(sampleOrder.id), Some(sampleOrder))
}
// sbt: testOnly -- --exclude-tags=slow   (fast local run)
// sbt: testOnly -- --include-tags=slow   (CI integration pass)
```

## Notes
- ScalaTest's equivalent is `object Slow extends org.scalatest.Tag("slow")` with `test("...", Slow) { ... }` in `AnyFunSuite`, or `"..." taggedAs Slow in { ... }` in `AnyWordSpec`.
- Keep the tag set small (`slow`, `integration`, `flaky`) — a tag per test defeats the point of grouping.
- A tag is not a substitute for fixing a genuinely flaky test; use it to separate tests by cost/dependency, not to hide known failures.

## References
- [MUnit — Tags](https://scalameta.org/munit/docs/tags.html)
- [ScalaTest — Tagging tests](https://www.scalatest.org/user_guide/tagging_your_tests)
