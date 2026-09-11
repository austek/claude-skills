---
title: Use forAll Instead of Hand-Rolled Input Loops
impact: MEDIUM
impactDescription: gets shrinking, seed reporting, and a configurable input count for free instead of reimplementing them
tags: [scalacheck, forall, generators]
---

# Use forAll Instead of Hand-Rolled Input Loops [MEDIUM]

## Description
When a test needs to check a property across many inputs, reach for ScalaCheck's `forAll`, not a manual `for`-loop over a hand-written list of cases or a `List.fill(n)(random...)`. `forAll` gives three things a loop can't replicate without reimplementing ScalaCheck itself: well-distributed random generation via `Gen`/`Arbitrary` (including boundary values, not just whatever the loop's author thought of), automatic shrinking of a failing case to a minimal counterexample (`scalacheck-shrinking-friendly-generators`), and a reported seed that reproduces the exact failing run. A manual loop over `Random.nextInt()` values gives none of these — a failure reports whichever iteration happened to fail, with no minimization and no reproducibility.

`forAll` also composes with multiple generated parameters directly (`forAll { (a: Int, b: Int) => ... }`), replacing what would otherwise be nested loops, and integrates with both MUnit (`munit.ScalaCheckSuite`) and ScalaTest (`org.scalatestplus.scalacheck.ScalaCheckPropertyChecks`) so the surrounding suite structure doesn't need to change.

## Bad Example
```scala
test("addition is commutative") {
  val random = scala.util.Random
  for _ <- 1 to 100 do
    val a = random.nextInt()
    val b = random.nextInt()
    assertEquals(a + b, b + a)
  // failure reports one unlucky pair, unminimized, and isn't reproducible on the next run
}
```

## Good Example
```scala
import org.scalacheck.Prop.forAll

class ArithmeticSuite extends munit.ScalaCheckSuite:
  property("addition is commutative") {
    forAll { (a: Int, b: Int) =>
      a + b == b + a
    }
  }
```

## Notes
- `forAll` bodies return `Boolean` or `Prop`; combine several checks with `&&` or `Prop.all(...)` rather than asserting mid-body with side effects.
- The default 100 generated cases is configurable per-suite via `override def scalaCheckTestParameters` (MUnit) or an implicit `PropertyCheckConfiguration` (ScalaTest) when a property needs more coverage.
- A logged seed from a CI failure can be pinned locally (`Test.Parameters.default.withInitialSeed(...)`) to deterministically reproduce the exact failing run.

## References
- [ScalaCheck — User Guide, Properties](https://github.com/typelevel/scalacheck/blob/main/doc/UserGuide.md#properties)
