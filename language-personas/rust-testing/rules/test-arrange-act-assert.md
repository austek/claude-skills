---
title: Structure Tests with Arrange, Act, Assert
impact: MEDIUM
impactDescription: Cuts time-to-diagnose on a failing test from minutes to seconds
tags: [testing, structure, readability, arrange-act-assert]
---

# Structure Tests with Arrange, Act, Assert [MEDIUM]

## Description
A test that mixes setup, invocation, and checks into one undifferentiated block forces the reader to reverse-engineer what's being verified every time it fails. Splitting a test into three labeled phases — build the inputs, call the thing under test once, check the outcome — turns the test itself into a spec of one behavior. When a test only asserts one thing, its failure message plus its name tell you exactly what broke without opening a debugger. Cramming several assertions covering different scenarios into a single test function is the opposite of this: one failing `assert_eq!` aborts the function before later assertions run, hiding whether the rest of the behavior is even correct.

## Bad Example
```rust
#[test]
fn test_user() {
    assert_eq!(User::new("alice", "alice@example.com").unwrap().name(), "alice");
    assert!(User::new("", "email@example.com").is_err());
    let u = User::new("bob", "bob@example.com").unwrap();
    assert!(u.validate());
    assert_eq!(u.email(), "bob@example.com");
}
```

## Good Example
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

#[test]
fn user_creation_fails_with_empty_name() {
    // Arrange
    let name = "";
    let email = "email@example.com";

    // Act
    let result = User::new(name, email);

    // Assert
    assert!(matches!(result, Err(UserError::EmptyName)));
}
```

## Notes
- One behavior per test function is the real goal; the AAA labels are just a way to enforce it. If a function needs more than one `// Act`, it's usually testing two behaviors and should be split.
- Extract repeated setup into small helper functions (`create_test_user()`, `create_order_with_items(&[...])`) rather than copy-pasting the Arrange block across tests — that keeps each test's Arrange section short enough to scan in a glance.
- For async code under `#[tokio::test]`, the same three-part shape still applies; `.await` just sits inside the Act step.
- Complex Arrange sections (building a multi-document search index, seeding several related structs) are a signal the type under test may have too many collaborators — consider it a design smell, not just a testing inconvenience.

## References
- [test-descriptive-names](test-descriptive-names.md)
- [test-fixture-raii](test-fixture-raii.md)
- [test-mock-traits](test-mock-traits.md)
