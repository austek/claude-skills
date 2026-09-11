---
title: Put Integration Tests in the tests/ Directory
impact: MEDIUM
impactDescription: Verifies the crate's real public surface instead of its internal shortcuts
tags: [testing, integration-tests, project-structure, public-api]
---

# Put Integration Tests in the tests/ Directory [MEDIUM]

## Description
Cargo treats every `.rs` file directly under `tests/` at the crate root as its own separate crate, compiled and linked against your library exactly the way an external consumer would depend on it — through `use my_crate::...` and nothing else. That constraint is the point: a test placed here can only exercise what's actually `pub`, so it catches the case where an internal refactor accidentally breaks the public contract even though every `#[cfg(test)]` unit test inside `src/` still passes (those have privileged access to internals and can't see that gap). Integration tests are therefore the layer that answers "does this crate still work for someone who only has the docs," distinct from unit tests answering "does this one function still do the right thing."

## Bad Example
```rust
// src/lib.rs — an integration-style test hiding inside the library crate,
// where it has (and can rely on) access to private internals it shouldn't need.
#[test]
fn integration_test_full_workflow() {
    // wrong location for a test meant to validate the public API
}
```

## Good Example
```rust
// tests/integration_test.rs
use my_crate::{Client, Config}; // only the public API is reachable here

#[test]
fn full_workflow_succeeds_with_default_config() {
    let client = Client::new(Config::default());
    let result = client.process("input");
    assert!(result.is_ok());
}

#[test]
fn strict_config_rejects_invalid_input() {
    let client = Client::new(Config::strict());
    let result = client.process("invalid");
    assert!(matches!(result, Err(Error::InvalidInput { .. })));
}
```

## Notes
- Shared setup code across multiple integration test files goes in `tests/common/mod.rs` — the `mod.rs` filename (rather than `common.rs`) tells Cargo this is a shared module, not another independent test binary to compile and run on its own.
- Larger suites can nest a directory of files under `tests/`, e.g. `tests/api/auth.rs` and `tests/api/users.rs`, wired together through a `tests/api/mod.rs` that declares them as submodules.
- Run just the integration layer with `cargo test --test '*'`, or a single file with `cargo test --test integration_test`; `cargo test --lib` runs only the in-crate unit tests, which is useful when iterating quickly on internals.
- Because every file in `tests/` is its own compiled crate, a workspace with many such files pays a real incremental-build cost — keep integration tests focused on cross-module behavior and leave fine-grained cases to unit tests.

## References
- [test-cfg-test-module](test-cfg-test-module.md)
- [test-descriptive-names](test-descriptive-names.md)
- [test-tokio-async](test-tokio-async.md)
