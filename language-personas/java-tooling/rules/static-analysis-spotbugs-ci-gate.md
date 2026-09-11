---
title: Fail the Build on SpotBugs Findings, Don't Just Report Them
impact: HIGH
impactDescription: turns bytecode-level bug patterns into a build-blocking gate instead of an ignored report
tags: [spotbugs, static-analysis, ci, bug-patterns]
---

# Fail the Build on SpotBugs Findings, Don't Just Report Them [HIGH]

## Description
SpotBugs (the maintained successor to FindBugs) analyzes compiled bytecode for roughly 400 known bug patterns grouped by category — correctness, bad practice, performance, security, multithreaded correctness — and ranked by confidence into Scariest/Scary/Troubling/Of Concern buckets. Because it inspects bytecode rather than source, it catches things a purely syntactic linter can't reliably see: inconsistent synchronization across a class's fields, a dropped return value from a method that must be checked (`RV_RETURN_VALUE_IGNORED`), or a null dereference reachable only through a specific control-flow path the compiler itself doesn't track.

Generating a SpotBugs HTML report without wiring it into the build's pass/fail outcome is close to worthless for maintaining a standard: reports nobody is forced to open get ignored, and findings accumulate silently. The gate has to be real — `check` depending on `spotbugsMain`/`spotbugsTest`, `ignoreFailures = false` — so a finding blocks the same build step a failing test would.

Adopting this on an existing codebase means starting with an `excludeFilter.xml` baseline that suppresses existing findings and gates only on *new* ones; demanding the full historical backlog be cleared before the gate can turn on is how teams end up disabling it instead.

## Bad Example
```kotlin
spotbugs {
    ignoreFailures = true // findings are generated, nothing ever fails
}
```

## Good Example
```kotlin
spotbugs {
    ignoreFailures = false
    excludeFilter = file("config/spotbugs/exclude.xml") // baseline, not a blanket suppression
}
tasks.check {
    dependsOn(tasks.spotbugsMain, tasks.spotbugsTest)
}
```

## Notes
- SpotBugs is distinct from PMD (source-based, style- and complexity-focused) and Error Prone (compile-time, javac-integrated) — the three catch overlapping but non-identical issue classes and are commonly run together.
- Prefer a targeted `@SuppressFBWarnings("BUG_CODE")` with a justification comment over adding a whole class to the exclude filter, so a suppression covers exactly the finding it's meant to and nothing else added later in that class.
- SpotBugs' `DE_MIGHT_IGNORE`/`REC_CATCH_EXCEPTION` patterns mechanically flag the same swallowed-exception shape covered in `../../java-coding-standards/rules/error-handling-no-swallowed-exceptions.md`.

## References
- [SpotBugs Manual](https://spotbugs.readthedocs.io/en/stable/)
- [SpotBugs Gradle Plugin](https://plugins.gradle.org/plugin/com.github.spotbugs)
