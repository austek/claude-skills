---
title: Treat .unwrap() in Non-Test Code as a Question, Not an Answer
impact: HIGH
impactDescription: Every uncaught None/Err becomes an unhandled crash instead of a recoverable path
tags: [anti-pattern, error-handling, panics, code-smell]
---

# Treat .unwrap() in Non-Test Code as a Question, Not an Answer [HIGH]

## Description
`.unwrap()` is the fastest way to make an `Option<T>`/`Result<T, E>` compile down to a `T` — which is also exactly the problem: it's easy to type without stopping to ask whether `None`/`Err` is actually impossible here, and the compiler enforces nothing about that assumption being correct. In production code, a wrong assumption becomes a runtime panic with a generic message (`called 'Option::unwrap()' on a 'None' value`) that says nothing about *why* the value was missing. The habit worth building is reading every `.unwrap()` as an implicit question — "can this actually be `None`/`Err` here?" — and only leaving it in place once that question has a confident, checkable "no."

## Bad Example
```rust
let content = std::fs::read_to_string("config.toml").unwrap(); // file might not exist
let value = map.get("key").unwrap();                            // key might be absent
let port: u16 = user_input.parse().unwrap();                    // input might be garbage
```

## Good Example
```rust
fn load_config() -> Result<Config, Error> {
    let content = std::fs::read_to_string("config.toml")?; // propagate, don't assume
    Ok(toml::from_str(&content)?)
}

let value = map.get("key").ok_or(Error::MissingKey)?;
let port: u16 = user_input.parse().map_err(|_| Error::InvalidPort)?;
```

## Notes
- Tests are the one place `.unwrap()` is unambiguously fine — a panicking assertion is the correct failure mode for a test, not something to route through `Result` handling.
- `.expect("message")` is a small, real improvement over bare `.unwrap()` when a panic genuinely is appropriate (see `anti-expect-lazy` for where that line actually sits) — but it's not a substitute for `Result` on a path that can fail in ordinary operation.
- `#![warn(clippy::unwrap_used)]` at the crate level, with narrow `#[allow(clippy::unwrap_used)]` overrides on files that are genuinely test-only, catches new instances of this before they reach review rather than relying on a human to spot each one.

## References
- [err-question-mark](err-question-mark.md)
- [err-result-over-panic](err-result-over-panic.md)
- [anti-expect-lazy](anti-expect-lazy.md)
