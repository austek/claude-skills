---
title: Treat Compiler Warnings as Errors in CI
impact: HIGH
impactDescription: stops warnings from accumulating into a backlog that buries the one new warning that matters
tags: [ci, javac, warnings, quality-gates]
---

# Treat Compiler Warnings as Errors in CI [HIGH]

## Description
javac emits warnings for deprecated API use, unchecked casts, raw-type usage, and similar issues, but by default a warning doesn't fail the build — it prints to a log nobody is required to read, and over years accumulates into hundreds of warnings that bury the one new warning introduced by today's change. Once a codebase has a large existing warning count, nobody can tell at a glance whether a given commit added a new problem, because the signal is already drowned in noise. `-Werror` (`options.compilerArgs` in Gradle, `<compilerArgument>-Werror</compilerArgument>` in Maven) promotes every enabled warning to a hard compile error, so warnings can never silently accumulate again — each one gets fixed, or explicitly and visibly suppressed with `@SuppressWarnings("deprecation")` at the point it's introduced, right when a reviewer can evaluate whether the suppression is justified.

`-Xlint:all` (or an explicit category list, `-Xlint:deprecation,unchecked,rawtypes`) controls which warning categories javac even emits in the first place; combined with `-Werror`, this is how a team dials strictness to exactly what it wants enforced, rather than accepting javac's default (fairly quiet) warning behavior.

## Bad Example
```kotlin
tasks.withType<JavaCompile>().configureEach {
    // no -Xlint, no -Werror — warnings print to a log nobody reads
}
```

## Good Example
```kotlin
tasks.withType<JavaCompile>().configureEach {
    options.compilerArgs.addAll(listOf("-Xlint:all", "-Werror"))
}
```

## Notes
- A legacy codebase with an existing warning backlog needs the same baseline strategy as `static-analysis-spotbugs-ci-gate.md` and `static-analysis-archunit-boundary-tests.md`: fix or suppress the current backlog module by module rather than blocking all work until the whole codebase is clean.
- `-Werror` also catches Error Prone's own diagnostics, since Error Prone integrates through the same javac warning/error channel — see `static-analysis-error-prone-compiler-plugin.md`.
- Apply to both `compileJava` and `compileTestJava` — test code accumulates the same warning debt as production code and is just as often skipped when only production sources are gated.

## References
- [javac — Command-Line Options](https://docs.oracle.com/en/java/javase/21/docs/specs/man/javac.html)
- [OpenJDK — Xlint Warnings](https://docs.oracle.com/en/java/javase/21/docs/specs/man/javac.html#option-xlint)
