---
name: rust-testing
description: Rust test-writing best practices covering test structure, fixtures, mocking, property-based testing, and benchmarks. Use when writing, reviewing, or refactoring tests, adding test coverage, or working in a tests directory or #[cfg(test)] module.
paths:
  - "**/tests/**"
---

# Rust Testing

A collection of test-writing best practices for Rust, adapted from the leonardomso/rust-skills catalog (MIT licensed — see [NOTICE.md](NOTICE.md)). Designed for AI agents and LLMs to write readable, maintainable, and trustworthy tests.

Note: the `paths:` glob above matches files under a `tests/` directory but cannot match inline `#[cfg(test)]` modules embedded in ordinary `.rs` source files — this skill still applies to those, just without path-based auto-triggering.

## Categories

### Testing [MEDIUM]
Write trustworthy Rust tests with idiomatic structure, mocking, and property-based coverage.

| Rule | Description |
|------|-------------|
| [test-arrange-act-assert](rules/test-arrange-act-assert.md) | Structure tests with arrange, act, assert |
| [test-cfg-test-module](rules/test-cfg-test-module.md) | Keep unit tests in a #[cfg(test)] module |
| [test-criterion-bench](rules/test-criterion-bench.md) | Benchmark with Criterion, not manual timing |
| [test-descriptive-names](rules/test-descriptive-names.md) | Name tests by the behavior they verify |
| [test-doctest-examples](rules/test-doctest-examples.md) | Keep doc examples as executable doctests |
| [test-fixture-raii](rules/test-fixture-raii.md) | Clean up test fixtures with RAII, not manual teardown |
| [test-integration-dir](rules/test-integration-dir.md) | Put integration tests in the tests/ directory |
| [test-loom-concurrency](rules/test-loom-concurrency.md) | Model-Check concurrent code with Loom |
| [test-mock-traits](rules/test-mock-traits.md) | Depend on traits, not concrete types, to enable mocking |
| [test-mockall-mocking](rules/test-mockall-mocking.md) | Use Mockall to generate trait mocks |
| [test-proptest-properties](rules/test-proptest-properties.md) | Test invariants with property-based testing (proptest) |
| [test-should-panic](rules/test-should-panic.md) | Assert expected panics with #[should_panic] |
| [test-snapshot-testing](rules/test-snapshot-testing.md) | Snapshot-Test large or structured output with Insta |
| [test-tokio-async](rules/test-tokio-async.md) | Drive async tests with #[tokio::test] |
| [test-use-super](rules/test-use-super.md) | Import parent items in tests with use super::* |

## Quick Reference

### Testing
```rust
#[test]
fn new_user_has_correct_name() {
    // Arrange
    let name = "alice";
    let email = "alice@example.com";

    // Act
    let user = User::new(name, email).unwrap();

    // Assert
    assert_eq!(user.name(), "alice");
}
```

## See Also

- [rust-coding-standards](../rust-coding-standards/SKILL.md) - General Rust coding standards and best practices
- [rust-tooling](../rust-tooling/SKILL.md) - clippy, rustfmt, and Cargo workspace/project-structure rules
- [NOTICE](NOTICE.md) - MIT attribution for content adapted from leonardomso/rust-skills
