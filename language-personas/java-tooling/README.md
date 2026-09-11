# Java Tooling

A comprehensive guide to Java development tools, designed for AI agents and LLMs to configure and use modern Java tooling effectively.

## Overview

This skill provides 20 rules across 5 categories:

| Category | Prefix | Impact | Rules |
|----------|--------|--------|-------|
| Build Configuration | `build-` | HIGH | 4 |
| Static Analysis | `static-analysis-` | HIGH | 4 |
| CI Quality Gates | `ci-quality-gates-` | MEDIUM | 4 |
| Dependency Management | `dependency-management-` | MEDIUM | 4 |
| Lint & Formatting | `lint-format-` | MEDIUM | 4 |

## Structure

```
skills/java-tooling/
├── SKILL.md              # Skill overview with all rule summaries
├── metadata.json         # Metadata (version, description)
├── README.md             # This file
└── rules/
    ├── _sections.md      # Section definitions
    ├── _template.md      # Rule template
    ├── build-*.md          # Build Configuration rules
    ├── static-analysis-*.md # Static Analysis rules
    ├── ci-quality-gates-*.md # CI Quality Gates rules
    ├── dependency-management-*.md # Dependency Management rules
    └── lint-format-*.md    # Lint & Formatting rules
```

## Rules

### Build Configuration (HIGH)
- `build-api-vs-implementation-scope` - Treat api vs implementation as a compile-avoidance lever, not just encapsulation
- `build-gradle-version-catalogs` - Centralize dependency versions in a Gradle version catalog
- `build-maven-dependency-management-bom` - Import a BOM in dependencyManagement instead of pinning every version
- `build-reproducible-builds` - Ban dynamic version ranges and non-deterministic archive output

### Static Analysis (HIGH)
- `static-analysis-archunit-boundary-tests` - Enforce package and layer boundaries with ArchUnit tests
- `static-analysis-error-prone-compiler-plugin` - Run error prone as a javac plugin, not a separate analysis pass
- `static-analysis-nullaway` - Enforce default non-null with NullAway, not just Objects.requireNonNull
- `static-analysis-spotbugs-ci-gate` - Fail the build on SpotBugs findings, don't just report them

### CI Quality Gates (MEDIUM)
- `ci-quality-gates-build-cache` - Enable a shared remote build cache so CI time tracks what changed
- `ci-quality-gates-coverage-threshold` - Gate CI on a deliberately scoped coverage threshold, not a blunt global percentage
- `ci-quality-gates-fail-on-warnings` - Treat compiler warnings as errors in CI
- `ci-quality-gates-mutation-testing` - Use PITest to verify tests actually assert something, not just execute lines

### Dependency Management (MEDIUM)
- `dependency-management-lockfile-or-bom` - Pick one source of truth — dependency lockfile or BOM, not neither
- `dependency-management-minimal-transitive-exposure` - Exclude unneeded transitive dependencies instead of accepting the full graph
- `dependency-management-version-catalog-single-source` - Make the version catalog the only place a version number can appear
- `dependency-management-vulnerability-scanning` - Scan dependencies for known CVEs on every build, not at release time

### Lint & Formatting (MEDIUM)
- `lint-format-checkstyle-config` - Scope Checkstyle to structural checks, not formatting
- `lint-format-google-java-format` - Adopt google-java-format for zero-config, deterministic formatting
- `lint-format-import-order` - Enforce deterministic import order, ban wildcard imports
- `lint-format-spotless-apply-check` - Gate CI on spotlessCheck, reserve spotlessApply for local hooks

## Related

- [java-coding-standards](../java-coding-standards/README.md) - General Java coding standards and best practices
- [java-testing](../java-testing/README.md) - Test-writing best practices for JUnit 5, AssertJ, Mockito, and Testcontainers

## Usage

This skill is automatically applied when working with Java files (`**/*.java`) and Java build files (`pom.xml`, `build.gradle`, `build.gradle.kts`).
