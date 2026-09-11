---
title: Gate CI on a Deliberately Scoped Coverage Threshold, Not a Blunt Global Percentage
impact: MEDIUM
impactDescription: catches untested new code in the module that changed, instead of letting the aggregate hide it
tags: [jacoco, coverage, ci, quality-gates]
---

# Gate CI on a Deliberately Scoped Coverage Threshold, Not a Blunt Global Percentage [MEDIUM]

## Description
JaCoCo instruments bytecode to measure which lines and branches actually executed during a test run, and its `jacocoTestCoverageVerification` task can fail the build when coverage falls below a configured rule — repo-wide, per-package, or per-class. The threshold is a regression floor, not a proxy for test quality: 100% line coverage is achievable with a test that calls every line and asserts nothing, so a coverage gate passing should never be read as "well tested" on its own. It's a cheap, mechanically gatable signal — necessary, not sufficient.

A single global percentage across a large codebase is a blunt instrument: it lets a handful of thoroughly-tested legacy modules pull the aggregate up while a brand-new, entirely untested module ships underneath that average, which is exactly the gap the gate was meant to catch. Scoping the rule per-package, or to diff coverage (only lines changed in this PR must meet the bar), targets enforcement at what's actually at risk in a given change, and lets a known low-coverage legacy module stay below the bar without blocking unrelated work — as long as nothing makes it worse.

## Bad Example
```kotlin
jacocoTestCoverageVerification {
    violationRules {
        rule { limit { minimum = "0.80".toBigDecimal() } } // one repo-wide average
    }
}
```
A new, untested payment module ships because three old, well-tested modules pull the aggregate above 80%.

## Good Example
```kotlin
jacocoTestCoverageVerification {
    violationRules {
        rule {
            element = "PACKAGE"
            includes = listOf("com.example.payments.*") // new/critical package held to a real bar
            limit { minimum = "0.90".toBigDecimal() }
        }
    }
}
```

## Notes
- Coverage measures "was this line executed," not "was this behavior verified" — pair it with `ci-quality-gates-mutation-testing.md` for the second half of that picture, since mutation testing catches exactly the assertion-free tests that inflate coverage numbers.
- `jacocoTestReport` produces the report; `jacocoTestCoverageVerification` is the separate task that turns it into a pass/fail gate — generating the report alone enforces nothing, the same gap `static-analysis-spotbugs-ci-gate.md` describes for an unwired SpotBugs report.
- Any legacy-module exemption from the threshold should be visible in the gate's own configuration (an explicit `excludes` list), not an informal understanding — an invisible exemption is indistinguishable from the gate silently not working.

## References
- [JaCoCo — Gradle Plugin](https://docs.gradle.org/current/userguide/jacoco_plugin.html)
- [JaCoCo](https://www.jacoco.org/jacoco/)
