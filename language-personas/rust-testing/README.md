# Rust Testing

Test-writing best practices for Rust, adapted from the leonardomso/rust-skills catalog for AI agents and LLMs to write readable, maintainable, and trustworthy tests.

## Overview

This skill provides 15 rules across 1 category:

| Category | Prefix | Impact | Rules |
|----------|--------|--------|-------|
| Testing | `test-` | MEDIUM | 15 |

## Structure

```
skills/rust-testing/
├── SKILL.md              # Skill overview with quick reference
├── metadata.json          # Metadata (version, description)
├── NOTICE.md              # MIT attribution to leonardomso/rust-skills
├── README.md              # This file
└── rules/
    ├── _sections.md      # Section definitions
    ├── _template.md      # Rule template
    └── test-*.md        # Testing rules
```

## Rules

### Testing (MEDIUM)
- `test-arrange-act-assert` - Structure tests with arrange, act, assert
- `test-cfg-test-module` - Keep unit tests in a #[cfg(test)] module
- `test-criterion-bench` - Benchmark with Criterion, not manual timing
- `test-descriptive-names` - Name tests by the behavior they verify
- `test-doctest-examples` - Keep doc examples as executable doctests
- `test-fixture-raii` - Clean up test fixtures with RAII, not manual teardown
- `test-integration-dir` - Put integration tests in the tests/ directory
- `test-loom-concurrency` - Model-Check concurrent code with Loom
- `test-mock-traits` - Depend on traits, not concrete types, to enable mocking
- `test-mockall-mocking` - Use Mockall to generate trait mocks
- `test-proptest-properties` - Test invariants with property-based testing (proptest)
- `test-should-panic` - Assert expected panics with #[should_panic]
- `test-snapshot-testing` - Snapshot-Test large or structured output with Insta
- `test-tokio-async` - Drive async tests with #[tokio::test]
- `test-use-super` - Import parent items in tests with use super::*

## Related

- [rust-coding-standards](../rust-coding-standards/README.md) - General Rust coding standards and best practices
- [rust-tooling](../rust-tooling/README.md) - Project structure and Clippy/rustc linting rules

## Usage

This skill is automatically applied when working under a `tests/` directory (`**/tests/**`). It also applies to inline `#[cfg(test)]` modules, which the `paths:` glob cannot match — apply it manually there.
