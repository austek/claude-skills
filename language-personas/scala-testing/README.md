# Scala Testing

Test-writing best practices for MUnit, ScalaTest, and ScalaCheck, designed for AI agents and LLMs to write readable, maintainable, and trustworthy tests.

## Overview

This skill provides 22 rules across 5 categories:

| Category | Prefix | Impact | Rules |
|----------|--------|--------|-------|
| Async Testing | `async-testing-` | HIGH | 4 |
| Mocking Discipline | `mocking-discipline-` | HIGH | 4 |
| ScalaCheck | `scalacheck-` | MEDIUM | 5 |
| ScalaTest & MUnit Structure | `scalatest-munit-structure-` | MEDIUM | 5 |
| Fixtures | `fixtures-` | MEDIUM | 4 |

## Structure

```
skills/scala-testing/
├── SKILL.md                        # Skill overview with quick reference
├── metadata.json                   # Metadata (version, description)
├── README.md                       # This file
└── rules/
    ├── _sections.md                # Section definitions
    ├── _template.md                # Rule template
    ├── async-testing-*.md          # Async Testing rules
    ├── mocking-discipline-*.md     # Mocking Discipline rules
    ├── scalacheck-*.md             # ScalaCheck rules
    ├── scalatest-munit-structure-*.md # ScalaTest & MUnit Structure rules
    └── fixtures-*.md               # Fixtures rules
```

## Rules

### Async Testing (HIGH)
- `async-testing-avoid-thread-sleep` - Never use Thread.sleep to wait for async work
- `async-testing-deterministic-clocks` - Test time-dependent code with a simulated clock
- `async-testing-future-recovertosucceededif` - Assert expected Future failures with recoverToSucceededIf
- `async-testing-io-testrunner` - Let CatsEffectSuite run IO tests instead of calling unsafeRunSync yourself

### Mocking Discipline (HIGH)
- `mocking-discipline-avoid-mockito-for-pure-fp` - Avoid Mockito on pure, effect-typed code
- `mocking-discipline-in-memory-interpreters` - Write in-memory interpreters for repository and client traits
- `mocking-discipline-mock-external-boundaries-only` - Reserve mocks for the external system boundary
- `mocking-discipline-prefer-fakes-over-mocks` - Prefer fakes over mocks

### ScalaCheck (MEDIUM)
- `scalacheck-arbitrary-instances-for-domain-types` - Provide Arbitrary instances for domain types
- `scalacheck-forall-over-manual-loops` - Use forAll instead of hand-rolled input loops
- `scalacheck-generator-composition` - Compose generators from smaller generators
- `scalacheck-property-based-for-pure-functions` - Reach for property-based tests on pure functions
- `scalacheck-shrinking-friendly-generators` - Keep generators shrinking-friendly

### ScalaTest & MUnit Structure (MEDIUM)
- `scalatest-munit-structure-aaa-pattern` - Structure test bodies as arrange-act-assert
- `scalatest-munit-structure-nested-suites` - Nest suites by shared context, not by file convenience
- `scalatest-munit-structure-one-behavior-per-test` - Assert one behavior per test
- `scalatest-munit-structure-suite-naming` - Name test suites after the unit under test
- `scalatest-munit-structure-tagged-tests` - Tag slow or environment-dependent tests

### Fixtures (MEDIUM)
- `fixtures-factory-methods-for-test-data` - Build test data with factory methods, not copy-pasted literals
- `fixtures-munit-lifecycle` - Prefer FunFixture over mutable beforeEach/afterEach in MUnit
- `fixtures-narrowest-scope` - Scope fixtures to the narrowest level that needs them
- `fixtures-scalatest-beforeandafter-traits` - Match ScalaTest fixture traits to setup cost

## Related

- [scala-coding-standards](../scala-coding-standards/README.md) - General Scala coding standards and best practices
- [scala-tooling](../scala-tooling/README.md) - Build, compiler flags, Scalafix, Scalafmt, Scalastyle, and Wartremover rules

## Usage

This skill is automatically applied when working with test files (`**/*Spec.scala`, `**/*Test.scala`, `**/*Suite.scala`).
