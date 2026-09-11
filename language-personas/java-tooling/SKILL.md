---
name: java-tooling
description: Java development tooling configuration and best practices for Gradle/Maven builds, static analysis, CI quality gates, dependency management, and lint/formatting. Use when setting up a Java project, configuring build tools, static analyzers, or CI gates, or when editing pom.xml or build.gradle(.kts).
paths:
  - "**/*.java"
  - "pom.xml"
  - "build.gradle"
  - "build.gradle.kts"
---

# Java Tooling

A comprehensive guide to Java development tooling. Configuration best practices for build systems, static analysis, CI quality gates, dependency management, and formatting.

## Categories

### Build Configuration [HIGH]
Configure Gradle and Maven builds for correct dependency scoping and reproducibility.

| Rule | Description |
|------|-------------|
| [build-api-vs-implementation-scope](rules/build-api-vs-implementation-scope.md) | Treat api vs implementation as a compile-avoidance lever, not just encapsulation |
| [build-gradle-version-catalogs](rules/build-gradle-version-catalogs.md) | Centralize dependency versions in a Gradle version catalog |
| [build-maven-dependency-management-bom](rules/build-maven-dependency-management-bom.md) | Import a BOM in dependencyManagement instead of pinning every version |
| [build-reproducible-builds](rules/build-reproducible-builds.md) | Ban dynamic version ranges and non-deterministic archive output |

### Static Analysis [HIGH]
Catch defects and boundary violations at build time with static analysis tools.

| Rule | Description |
|------|-------------|
| [static-analysis-archunit-boundary-tests](rules/static-analysis-archunit-boundary-tests.md) | Enforce package and layer boundaries with ArchUnit tests |
| [static-analysis-error-prone-compiler-plugin](rules/static-analysis-error-prone-compiler-plugin.md) | Run error prone as a javac plugin, not a separate analysis pass |
| [static-analysis-nullaway](rules/static-analysis-nullaway.md) | Enforce default non-null with NullAway, not just Objects.requireNonNull |
| [static-analysis-spotbugs-ci-gate](rules/static-analysis-spotbugs-ci-gate.md) | Fail the build on SpotBugs findings, don't just report them |

### CI Quality Gates [MEDIUM]
Gate CI on build cache, coverage, warnings, and mutation testing so it enforces real quality.

| Rule | Description |
|------|-------------|
| [ci-quality-gates-build-cache](rules/ci-quality-gates-build-cache.md) | Enable a shared remote build cache so CI time tracks what changed |
| [ci-quality-gates-coverage-threshold](rules/ci-quality-gates-coverage-threshold.md) | Gate CI on a deliberately scoped coverage threshold, not a blunt global percentage |
| [ci-quality-gates-fail-on-warnings](rules/ci-quality-gates-fail-on-warnings.md) | Treat compiler warnings as errors in CI |
| [ci-quality-gates-mutation-testing](rules/ci-quality-gates-mutation-testing.md) | Use PITest to verify tests actually assert something, not just execute lines |

### Dependency Management [MEDIUM]
Keep dependency versions and transitive exposure under single-source control.

| Rule | Description |
|------|-------------|
| [dependency-management-lockfile-or-bom](rules/dependency-management-lockfile-or-bom.md) | Pick one source of truth — dependency lockfile or BOM, not neither |
| [dependency-management-minimal-transitive-exposure](rules/dependency-management-minimal-transitive-exposure.md) | Exclude unneeded transitive dependencies instead of accepting the full graph |
| [dependency-management-version-catalog-single-source](rules/dependency-management-version-catalog-single-source.md) | Make the version catalog the only place a version number can appear |
| [dependency-management-vulnerability-scanning](rules/dependency-management-vulnerability-scanning.md) | Scan dependencies for known CVEs on every build, not at release time |

### Lint & Formatting [MEDIUM]
Enforce consistent formatting and import order without relying on human review.

| Rule | Description |
|------|-------------|
| [lint-format-checkstyle-config](rules/lint-format-checkstyle-config.md) | Scope Checkstyle to structural checks, not formatting |
| [lint-format-google-java-format](rules/lint-format-google-java-format.md) | Adopt google-java-format for zero-config, deterministic formatting |
| [lint-format-import-order](rules/lint-format-import-order.md) | Enforce deterministic import order, ban wildcard imports |
| [lint-format-spotless-apply-check](rules/lint-format-spotless-apply-check.md) | Gate CI on spotlessCheck, reserve spotlessApply for local hooks |

## Quick Reference

### Build Configuration
```toml
# gradle/libs.versions.toml
[versions]
jackson = "2.17.0"

[libraries]
jackson-databind = { module = "com.fasterxml.jackson.core:jackson-databind", version.ref = "jackson" }

[bundles]
testing = ["junit-jupiter", "assertj-core", "mockito-core"]
```

### Static Analysis
```java
@AnalyzeClasses(packages = "com.example")
class ArchitectureTest {

    @ArchTest
    static final ArchRule domainStaysFree = noClasses()
        .that().resideInAPackage("..domain..")
        .should().dependOnClassesThat().resideInAnyPackage("..infrastructure..", "org.springframework..");
}
```

### CI Quality Gates
```kotlin
tasks.withType<JavaCompile>().configureEach {
    options.compilerArgs.addAll(listOf("-Xlint:all", "-Werror"))
}
```

### Dependency Management
```bash
# same PR that bumps the BOM version
./gradlew dependencies --write-locks
git add gradle.lockfile
```

### Lint & Formatting
```kotlin
tasks.check {
    dependsOn(tasks.spotlessCheck) // CI verifies, never rewrites
}
```

## See Also

- [java-coding-standards](../java-coding-standards/SKILL.md) - General Java coding standards and best practices
- [java-testing](../java-testing/SKILL.md) - Test-writing best practices for JUnit 5, AssertJ, Mockito, and Testcontainers
