---
title: Assert Expected Panics with #[should_panic]
impact: LOW
impactDescription: Replaces a manual catch_unwind dance with a single declarative attribute
tags: [testing, panic, should-panic, invariants]
---

# Assert Expected Panics with #[should_panic] [LOW]

## Description
Code that enforces an invariant by panicking — an assertion inside a constructor, an out-of-bounds index — needs a way to verify the panic actually fires without the panic itself failing the test run. `#[should_panic]` does exactly that: the test framework catches the unwind, and the test *passes* precisely when the panic happens. Adding `expected = "..."` tightens the check to confirm the panic message contains a specific substring, guarding against a test that "passes" only because some unrelated panic fired earlier than the one actually being verified.

## Bad Example
```rust
#[test]
fn test_panic_manual() {
    // Works, but is verbose compared to the attribute, and callers tend to
    // skip the substring check this way, weakening what the test proves.
    let result = std::panic::catch_unwind(|| divide(1, 0));
    assert!(result.is_err());
}
```

## Good Example
```rust
#[test]
#[should_panic(expected = "division by zero")]
fn divide_by_zero_panics_with_message() {
    divide(1, 0);
}

struct NonEmpty<T>(Vec<T>);

impl<T> NonEmpty<T> {
    fn new(items: Vec<T>) -> Self {
        assert!(!items.is_empty(), "NonEmpty cannot be empty");
        NonEmpty(items)
    }
}

#[test]
#[should_panic(expected = "NonEmpty cannot be empty")]
fn non_empty_rejects_empty_vec() {
    NonEmpty::new(Vec::<i32>::new());
}
```

## Notes
- The `expected` string only needs to be a substring of the actual panic message, not an exact match — this keeps the test resilient to minor wording changes while still confirming the right panic path was hit.
- Reserve `#[should_panic]` for genuinely unrecoverable, programmer-error conditions. If the failure is something a caller should be able to handle — invalid user input, a malformed file — the function should return `Result` and the test should assert on `is_err()` instead; using panic for recoverable errors pushes error handling out of the type system.
- A `#[should_panic]` test can still return a `Result` from setup code that uses `?`, as long as the actual panic happens before the function would otherwise return `Ok(())` — the `Ok(())` branch is simply never reached in a passing test.
- Without `expected`, the test passes on *any* panic in the function body, including one from an unrelated bug earlier in the setup — prefer the message-checked form whenever the panic message is stable enough to assert on.

## References
- [err-result-over-panic](../../rust-coding-standards/rules/err-result-over-panic.md)
- [err-expect-bugs-only](../../rust-coding-standards/rules/err-expect-bugs-only.md)
- [test-descriptive-names](test-descriptive-names.md)
