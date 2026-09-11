---
title: Use Mockall to Generate Trait Mocks
impact: MEDIUM
impactDescription: Removes the boilerplate of hand-writing a fake implementation for every mocked trait
tags: [testing, mockall, mocking, traits]
---

# Use Mockall to Generate Trait Mocks [MEDIUM]

## Description
Writing a mock by hand — a struct that stores its own call log, matches arguments manually, and returns canned values — is straightforward for one method but turns tedious fast once a trait has several methods, argument matching, or call-count requirements. `mockall`'s `#[automock]` attribute generates a `MockYourTrait` type from the trait definition itself, with a fluent `expect_*` API for specifying which calls are expected, what arguments they should receive, how many times they should happen, and what to return — all verified automatically when the mock is dropped at the end of the test. It's the same trait-based testability as writing your own fake, minus the boilerplate of maintaining that fake by hand as the trait evolves.

## Bad Example
```rust
// Hand-rolled mock: correct but grows unwieldy once you need call counts,
// argument matching, and ordering checks across several methods.
struct ManualMockDatabase {
    calls: std::cell::RefCell<Vec<u64>>,
}

impl Database for ManualMockDatabase {
    fn get_user(&self, id: u64) -> Option<User> {
        self.calls.borrow_mut().push(id);
        Some(User { id, name: "Alice".into() })
    }
}
```

## Good Example
```rust
use mockall::automock;

#[automock]
trait Database {
    fn get_user(&self, id: u64) -> Option<User>;
}

#[cfg(test)]
mod tests {
    use super::*;
    use mockall::predicate::*;

    #[test]
    fn find_user_returns_name_from_repository() {
        let mut mock = MockDatabase::new();
        mock.expect_get_user()
            .with(eq(42))
            .times(1)
            .returning(|_| Some(User { id: 42, name: "Alice".into() }));

        let service = UserService::new(mock);
        assert_eq!(service.find_user(42).unwrap().name, "Alice");
        // Mockall panics on drop if expect_get_user() was never called.
    }
}
```

## Notes
- `.times(1)`, `.times(3..)`, or a bare call with no `.times()` (any number of calls) all constrain how many invocations are acceptable; mockall checks these expectations when the mock is dropped, not when you set them up, so a missed call fails the test even without an explicit assertion.
- `mockall::Sequence` lets you assert that calls across several `expect_*` clauses happen in a specific order — useful for verifying a connect-query-disconnect lifecycle rather than just that each call happened at some point.
- `.with(eq(42))` matches an exact value; `.withf(|arg| ...)` accepts an arbitrary predicate closure for cases an exact match can't express, such as checking a string's length or a struct field's range.
- For traits you don't own (from another crate), `#[cfg_attr(test, mockall::automock)]` applies the macro only during test compilation, so the mocked version never affects the production build.

## References
- [test-mock-traits](test-mock-traits.md)
- [test-proptest-properties](test-proptest-properties.md)
- [test-arrange-act-assert](test-arrange-act-assert.md)
