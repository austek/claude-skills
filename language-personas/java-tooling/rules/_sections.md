# Section Definitions

## Build Configuration (build)
**Impact:** HIGH

Configure Gradle and Maven builds for correct dependency scoping and reproducibility.

**Rules:**
- `build-api-vs-implementation-scope` - Treat api vs implementation as a compile-avoidance lever, not just encapsulation
- `build-gradle-version-catalogs` - Centralize dependency versions in a Gradle version catalog
- `build-maven-dependency-management-bom` - Import a BOM in dependencyManagement instead of pinning every version
- `build-reproducible-builds` - Ban dynamic version ranges and non-deterministic archive output

## Static Analysis (static-analysis)
**Impact:** HIGH

Catch defects and boundary violations at build time with static analysis tools.

**Rules:**
- `static-analysis-archunit-boundary-tests` - Enforce package and layer boundaries with ArchUnit tests
- `static-analysis-error-prone-compiler-plugin` - Run error prone as a javac plugin, not a separate analysis pass
- `static-analysis-nullaway` - Enforce default non-null with NullAway, not just Objects.requireNonNull
- `static-analysis-spotbugs-ci-gate` - Fail the build on SpotBugs findings, don't just report them

## CI Quality Gates (ci-quality-gates)
**Impact:** MEDIUM

Gate CI on build cache, coverage, warnings, and mutation testing so it enforces real quality.

**Rules:**
- `ci-quality-gates-build-cache` - Enable a shared remote build cache so CI time tracks what changed
- `ci-quality-gates-coverage-threshold` - Gate CI on a deliberately scoped coverage threshold, not a blunt global percentage
- `ci-quality-gates-fail-on-warnings` - Treat compiler warnings as errors in CI
- `ci-quality-gates-mutation-testing` - Use PITest to verify tests actually assert something, not just execute lines

## Dependency Management (dependency-management)
**Impact:** MEDIUM

Keep dependency versions and transitive exposure under single-source control.

**Rules:**
- `dependency-management-lockfile-or-bom` - Pick one source of truth — dependency lockfile or BOM, not neither
- `dependency-management-minimal-transitive-exposure` - Exclude unneeded transitive dependencies instead of accepting the full graph
- `dependency-management-version-catalog-single-source` - Make the version catalog the only place a version number can appear
- `dependency-management-vulnerability-scanning` - Scan dependencies for known CVEs on every build, not at release time

## Lint & Formatting (lint-format)
**Impact:** MEDIUM

Enforce consistent formatting and import order without relying on human review.

**Rules:**
- `lint-format-checkstyle-config` - Scope Checkstyle to structural checks, not formatting
- `lint-format-google-java-format` - Adopt google-java-format for zero-config, deterministic formatting
- `lint-format-import-order` - Enforce deterministic import order, ban wildcard imports
- `lint-format-spotless-apply-check` - Gate CI on spotlessCheck, reserve spotlessApply for local hooks
