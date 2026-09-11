---
title: Gate CI on spotlessCheck, Reserve spotlessApply for Local Hooks
impact: MEDIUM
impactDescription: an unformatted file fails the build like a failing test, instead of drifting in unreviewed
tags: [spotless, formatting, gradle, ci]
---

# Gate CI on spotlessCheck, Reserve spotlessApply for Local Hooks [MEDIUM]

## Description
Spotless is a Gradle/Maven plugin that wraps an underlying formatter — google-java-format, the Eclipse formatter, Palantir's formatter, or non-Java formatters like Prettier for YAML/Markdown in the same build — behind one consistent task pair: `spotlessApply` rewrites every file in place to the formatted result, and `spotlessCheck` compares each file against what `spotlessApply` would produce and fails the build if any file differs, without touching anything. Because formatting is fully deterministic, `spotlessCheck` is the task that belongs in CI — `check` should depend on it, so an unformatted file fails the build the same way a failing test does, catching drift before merge rather than relying on a reviewer to notice inconsistent whitespace.

`spotlessApply`, conversely, does not belong in CI. A CI pipeline that runs `spotlessApply` and pushes the result back mutates the branch out from under whoever pushed it, which is surprising at best and actively harmful if it interacts badly with a force-push or a signed-commit requirement. `spotlessApply` belongs in a local pre-commit hook or an editor save-action, where a developer sees and accepts the rewrite before it's committed — with that hook in place, `spotlessCheck` failures in CI become rare, because formatting drift never leaves the developer's machine.

## Bad Example
```kotlin
// CI pipeline step
./gradlew spotlessApply
git commit -am "auto-format" && git push // rewrites the branch mid-CI
```

## Good Example
```kotlin
tasks.check {
    dependsOn(tasks.spotlessCheck) // CI verifies, never rewrites
}
```
```bash
# .git/hooks/pre-commit (or a managed hook manager)
./gradlew spotlessApply
```

## Notes
- `ratchetFrom("origin/main")` scopes Spotless checks to files changed relative to a base branch, letting a large pre-existing unformatted codebase adopt Spotless without a single mass-reformat commit gating adoption.
- Spotless commonly wraps `lint-format-google-java-format.md`'s formatter for Java sources while formatting other file types in the same build config, avoiding a separate formatter dependency and task per language.
- A mass reformat commit (the one-time `spotlessApply` that first brings a codebase into compliance) should land as its own commit, isolated from any logic change, so `git blame` on surrounding lines still points at meaningful history.

## References
- [Spotless — Gradle Plugin](https://github.com/diffplug/spotless/tree/main/plugin-gradle)
- [Spotless — Maven Plugin](https://github.com/diffplug/spotless/tree/main/plugin-maven)
