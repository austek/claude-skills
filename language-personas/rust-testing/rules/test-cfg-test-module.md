---
title: Keep Unit Tests in a #[cfg(test)] Module
impact: HIGH
impactDescription: Strips test code and its test-only dependencies from release binaries entirely
tags: [testing, unit-tests, cfg-test, modules]
---

# Keep Unit Tests in a #[cfg(test)] Module [HIGH]

## Description
`#[cfg(test)]` is a conditional-compilation attribute — the compiler only emits the annotated module when building for `cargo test`, never for a normal `cargo build` or `cargo build --release`. Forgetting it means test functions, their assertions, and any test-only crates they pull in ride along inside the shipped binary, bloating it and occasionally leaking test-only behavior into production. The companion convention — nesting that module directly inside the file it tests, one level below the code — also solves a problem integration tests can't: `mod tests` sits as a child of the module it exercises, so `use super::*` reaches private functions and types that no external crate could ever see. That combination is why this is Rust's default unit-testing shape rather than a style preference.

## Bad Example
```rust
// Missing #[cfg(test)] — this module compiles into release builds too.
mod tests {
    #[test]
    fn test_something() { /* ... */ }
}

// src/my_module.rs
fn private_helper() -> i32 { 21 }

// tests/my_module_test.rs — can't reach private_helper from here at all.
```

## Good Example
```rust
// src/my_module.rs
fn public_api() -> i32 {
    private_helper() * 2
}

fn private_helper() -> i32 {
    21
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn public_api_doubles_helper_value() {
        assert_eq!(public_api(), 42);
    }

    #[test]
    fn private_helper_returns_21() {
        assert_eq!(private_helper(), 21);
    }
}
```

## Notes
- Because the module lives beside the code, moving or renaming a function and forgetting to update its test is an immediate compile error in the same file — there's no separate test tree to fall out of sync.
- For larger files, nest further: `mod tests { mod parsing { ... } mod validation { ... } }` groups related cases without needing separate files.
- This pattern is for *unit* tests that need access to internals. Tests that only exercise the crate's public surface belong in `tests/` instead — see the integration-tests rule for when to prefer that.
- Test-only helper functions defined inside the `tests` module (fixtures, assertion wrappers) are also compiled out of release builds automatically, since they live inside the same `#[cfg(test)]` boundary.

## References
- [test-use-super](test-use-super.md)
- [test-integration-dir](test-integration-dir.md)
- [test-descriptive-names](test-descriptive-names.md)
