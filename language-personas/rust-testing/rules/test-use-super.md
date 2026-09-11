---
title: Import Parent Items in Tests with use super::*
impact: LOW
impactDescription: Saves an explicit import line per item and gives tests access to private items in one statement
tags: [testing, imports, use-super, visibility]
---

# Import Parent Items in Tests with use super::* [LOW]

## Description
A `#[cfg(test)] mod tests` block is a child module of the code it tests, and Rust's module visibility rules mean a child can always see its parent's private items — no `pub` required. `use super::*` takes advantage of that in one line: it pulls in every item from the enclosing module, public or private, rather than requiring a hand-maintained list of individual imports that has to be updated whenever the test file starts exercising a new function or type. This is what lets a test call a private helper function directly and assert on its behavior, something an external integration test in `tests/` structurally cannot do.

## Bad Example
```rust
#[cfg(test)]
mod tests {
    use crate::my_module::public_function;
    use crate::my_module::MyStruct;
    // Explicit crate-path imports only reach pub items — private helpers
    // in the same file are unreachable this way even though they're
    // logically part of what this module is testing.

    #[test]
    fn test_function() {
        let result = public_function();
        // ...
    }
}
```

## Good Example
```rust
// src/my_module.rs
pub fn public_function() -> i32 { 42 }
fn private_helper() -> i32 { 42 }

#[cfg(test)]
mod tests {
    use super::*; // pulls in everything from the parent module

    #[test]
    fn private_helper_returns_42() {
        assert_eq!(private_helper(), 42); // only reachable via super
    }
}
```

## Notes
- `use super::{parse, ParseError, Token};` is the explicit alternative when you'd rather document exactly which parent items a test file depends on — reach for it in modules with a very large public surface where a blanket glob import would pull in far more than the tests actually use.
- In nested modules, `use super::*` reaches only the immediate parent; a `tests` module two levels deep needs `use super::super::*` (or an equivalent path) to reach the grandparent module's items as well.
- `use super::*` composes normally with other test-only imports — `use proptest::prelude::*;` or `use mockall::predicate::*;` sit alongside it without conflict, since they're importing from different crates entirely.
- This pattern is specific to the in-file `#[cfg(test)]` unit-test convention; files under `tests/` are separate crates with no `super` module to import from, since they only see the library's public API.

## References
- [test-cfg-test-module](test-cfg-test-module.md)
- [test-integration-dir](test-integration-dir.md)
- [proj-pub-crate-internal](../../rust-tooling/rules/proj-pub-crate-internal.md)
