---
title: Use PITest to Verify Tests Actually Assert Something, Not Just Execute Lines
impact: MEDIUM
impactDescription: reveals a passing, fully-covered test suite that would never notice a real behavioral regression
tags: [pitest, mutation-testing, ci, quality-gates]
---

# Use PITest to Verify Tests Actually Assert Something, Not Just Execute Lines [MEDIUM]

## Description
Line and branch coverage answers "did a test execute this line," a bar a test satisfies by calling the code and asserting nothing. PITest asks a stronger question: it systematically mutates the bytecode under test — flips a `>` to `>=`, negates a boolean condition, removes a method call, swaps a returned constant — and reruns the test suite against each mutant. A mutant that *survives*, meaning the suite still passes despite a genuine behavioral change, reveals a test that runs the line but never checks the specific outcome that mutation altered. The mutation score (percentage of mutants killed) is a far harder metric to game than a coverage percentage, precisely because it requires an assertion to actually fail when the underlying behavior changes.

Because PITest reruns the full relevant test suite once per surviving-candidate mutant, it's computationally far more expensive than a single coverage pass, so it typically runs less often than every commit — nightly, or scoped to PRs touching specific high-value modules — rather than gating every push the way a compiler-warning or coverage-threshold check does. Running it against everything on every commit is rarely worth the CI time it costs relative to the signal gained on low-risk modules.

## Bad Example
```java
@Test
void processesOrderWithoutThrowing() {
    assertDoesNotThrow(() -> service.process(order)); // 100% line coverage, kills zero mutants
}
```

## Good Example
```java
@Test
void processAppliesDiscountToEligibleOrder() {
    Order result = service.process(order);

    assertThat(result.total()).isEqualByComparingTo("90.00"); // fails if a mutant changes the calculation
}
```

## Notes
- A low mutation score on a well-covered class is a stronger signal of weak tests than a low coverage percentage on the same class — coverage says the code ran, mutation testing says the tests would notice if the code were wrong.
- "Equivalent mutants" — a mutation that's syntactically different but behaviorally identical, so no test could ever kill it — are a known source of noise in the score; review individual survivors rather than chasing 100%.
- Run mutation testing only where `ci-quality-gates-coverage-threshold.md`'s gate already passes — mutants inside uncovered code are neither killed nor meaningfully survived, they're simply never exercised, so the score there is uninformative.

## References
- [PITest](https://pitest.org/)
- [PITest — Gradle Plugin](https://plugins.gradle.org/plugin/info.solidsoft.pitest)
