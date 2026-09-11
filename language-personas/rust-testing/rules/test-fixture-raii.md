---
title: Clean Up Test Fixtures with RAII, Not Manual Teardown
impact: HIGH
impactDescription: Guarantees cleanup runs even when an assertion panics mid-test
tags: [testing, raii, drop, fixtures, cleanup]
---

# Clean Up Test Fixtures with RAII, Not Manual Teardown [HIGH]

## Description
A test that writes a temp file, sets an environment variable, or opens a connection and then cleans it up as the last line of the function has a hidden failure mode: if any assertion above that line panics, the cleanup call never runs. The next test in the suite — or a completely unrelated one, since `cargo test` runs tests in parallel by default — inherits leftover state and starts failing for reasons that have nothing to do with its own logic. Rust's `Drop` trait sidesteps this: a value's destructor runs during unwinding, not just on the normal return path, so tying a resource's lifetime to a guard struct's scope makes cleanup unconditional. This is the same RAII discipline C++ and Rust use for memory; applying it to test fixtures is what keeps a large suite from becoming order-dependent and flaky.

## Bad Example
```rust
#[test]
fn test_with_temp_file() {
    let path = "/tmp/test_file.txt";
    std::fs::write(path, "test data").unwrap();

    let result = process_file(path);

    std::fs::remove_file(path).unwrap(); // skipped entirely if the line above panics
    assert!(result.is_ok());
}
```

## Good Example
```rust
use tempfile::NamedTempFile;

#[test]
fn test_with_temp_file() {
    // Arrange — the file is deleted automatically when `file` goes out of scope,
    // including on an assertion panic below.
    let file = NamedTempFile::new().unwrap();
    std::fs::write(file.path(), "test data").unwrap();

    // Act
    let result = process_file(file.path());

    // Assert
    assert!(result.is_ok());
}
```

## Notes
- `tempfile::NamedTempFile` and `tempfile::TempDir` cover the two most common fixtures — a scratch file and a scratch directory — and both delete their contents on drop without any custom code.
- For anything the crate doesn't provide off the shelf (a spawned test server, a database transaction that must roll back), write a small guard struct whose `Drop` impl performs the teardown, and hold it as an unused-but-alive binding (`let _guard = ...;`) for the test's duration.
- The `scopeguard` crate's `defer! { ... }` macro is a lighter alternative to a custom guard type when the cleanup is a one-off block rather than something reusable across several tests.
- Environment-variable mutation is inherently global process state; a guard that restores the previous value on drop only makes tests safe if those tests aren't also running in parallel against the same variable — serialize such tests or isolate the variable per test where possible.

## References
- [test-arrange-act-assert](test-arrange-act-assert.md)
- [test-tokio-async](test-tokio-async.md)
- [test-mock-traits](test-mock-traits.md)
