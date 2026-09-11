---
title: Name Tests by the Behavior They Verify
impact: MEDIUM
impactDescription: Turns a red CI run into an immediate diagnosis instead of a research task
tags: [testing, naming, readability, documentation]
---

# Name Tests by the Behavior They Verify [MEDIUM]

## Description
`cargo test`'s output is a flat list of function names — that name is the only piece of information you get before you go digging into the test body. `test1` or `it_works` tells a reader nothing when it shows up under `FAILED`; `parse_returns_error_for_empty_input` tells the reader what broke before they've opened the file. A test name is effectively a one-line spec: read down the list of names in a module and you get a plain-language description of everything that module's behavior guarantees, which doubles as living documentation nobody has to remember to update separately.

## Bad Example
```rust
#[test]
fn test1() { /* ... */ }

#[test]
fn test_parse() { /* ... */ } // parse what, under what condition?

#[test]
fn it_works() { /* ... */ }
```

## Good Example
```rust
#[test]
fn parse_returns_error_for_empty_input() { /* ... */ }

#[test]
fn expired_token_is_rejected() { /* ... */ }

#[test]
fn adding_item_increases_cart_total() { /* ... */ }
```

## Notes
- A few naming shapes cover most cases and are worth picking consistently within a crate: `subject_condition_outcome` (`parse_invalid_json_returns_syntax_error`), `scenario_expectation` (`empty_cart_has_zero_total`), or BDD-flavored `when_x_then_y` for request-handling code.
- Naming edge cases explicitly (`handles_empty_string`, `handles_max_length_input`, `handles_concurrent_access`) doubles as an inventory of which edge cases are actually covered — a reviewer can scan test names to spot a missing case rather than reading every test body.
- Nesting tests in submodules (`mod parsing { ... } mod validation { ... }`) namespaces the output further, so `tests::parsing::accepts_valid_json` reads as a sentence even without repeating "parsing" in every individual test name.
- A name that just restates the function under test (`test_parse`) without saying which input or which outcome is being checked provides no more information than the file and line number already give you — it isn't documentation, it's a label.

## References
- [test-arrange-act-assert](test-arrange-act-assert.md)
- [test-cfg-test-module](test-cfg-test-module.md)
- [doc-examples-section](../../rust-coding-standards/rules/doc-examples-section.md)
