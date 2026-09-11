---
title: Enforce Package and Layer Boundaries with ArchUnit Tests
impact: HIGH
impactDescription: a forbidden import fails the build the same way a broken test does, instead of surviving review
tags: [archunit, static-analysis, architecture, testing]
---

# Enforce Package and Layer Boundaries with ArchUnit Tests [HIGH]

## Description
ArchUnit is a plain Java library, not an external tool with its own CLI — its rules are written as ordinary JUnit tests that use reflection over compiled classes to assert structural properties: `domain` code never imports `infrastructure`, classes annotated `@Service` live only in a `service` package, no package-cycle exists between two modules. Because the assertion runs as a test, a violation fails the exact same build step as any other broken test and shows up in the same CI report — unlike an architecture diagram or a wiki page describing "the rules," which doesn't fail anything when someone adds a forbidden import six months after the diagram was drawn and nobody re-reads it.

This complements module-system boundaries rather than replacing them. JPMS (`../../java-coding-standards/rules/modules-jpms-explicit-exports.md`) enforces what's declared in `module-info.java` at compile and run time; ArchUnit asserts finer-grained or cross-cutting rules that don't map cleanly onto module boundaries — no framework annotation inside the domain layer, no package residing in two different logical layers simultaneously — and works identically in codebases that haven't adopted the module system at all, since it only needs compiled classes on a test classpath.

The `layeredArchitecture()` DSL describes a whole application's layering (`domain`, `application`, `infrastructure`, `web`) and their allowed dependency directions in one rule, which tends to catch more violations with less test-writing effort than accumulating many narrow `noClasses().should()...` rules one at a time.

## Bad Example
```java
// domain/OrderService.java — no test catches this until someone notices in review
package com.example.domain;

import org.springframework.stereotype.Service; // framework leaking into domain
import com.example.infrastructure.JpaOrderRepository; // layer violation
```

## Good Example
```java
@AnalyzeClasses(packages = "com.example")
class ArchitectureTest {

    @ArchTest
    static final ArchRule domainStaysFree = noClasses()
        .that().resideInAPackage("..domain..")
        .should().dependOnClassesThat().resideInAnyPackage("..infrastructure..", "org.springframework..");
}
```

## Notes
- `freeze()` locks in a snapshot of current (known) violations as a baseline and fails only on *new* ones, letting a legacy codebase adopt ArchUnit without a mass cleanup gating adoption — the same strategy `static-analysis-spotbugs-ci-gate.md` uses for bug-pattern baselining.
- Rules run inside the normal `test` task via the standard ArchUnit JUnit 5 integration — no separate plugin or CI task to wire in beyond adding the dependency.
- `slices().matching(...)` detects package cycles, a class of bug that's otherwise invisible until a `module-info.java` migration or a build-tool circular-dependency error forces the issue.

## References
- [ArchUnit User Guide](https://www.archunit.org/userguide/html/000_Index.html)
- [ArchUnit — GitHub](https://github.com/TNG/ArchUnit)
