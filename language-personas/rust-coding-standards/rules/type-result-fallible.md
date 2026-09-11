---
title: Use Result for Operations That Can Fail
impact: CRITICAL
impactDescription: Forces callers to acknowledge failure instead of silently ignoring it
tags: [type-safety, result, error-handling, question-mark]
---

# Use Result for Operations That Can Fail [CRITICAL]

## Description
`Result<T, E>` puts failure into the type system: a caller cannot get at the `T` without going through `Ok`/`Err`, so ignoring a failure requires an explicit, visible choice (`.unwrap()`, `.ok()`) rather than an accidental oversight. Returning `Option<T>` for a fallible operation discards *why* it failed, and returning a sentinel value (`-1` for "division by zero") relies on every caller remembering the magic number's meaning. `Result` keeps the reason attached to the failure, and the `?` operator makes propagating it as terse as ignoring it would have been — so there's no ergonomic tax for doing it correctly.

## Bad Example
```rust
// Sentinel value: easy to miss, no explanation of what happened.
fn divide(a: i32, b: i32) -> i32 {
    if b == 0 { return -1; }
    a / b
}

// Panicking on errors instead of returning them.
fn read_config(path: &str) -> Config {
    let content = std::fs::read_to_string(path).unwrap(); // crashes
    toml::from_str(&content).unwrap()
}
```

## Good Example
```rust
fn divide(a: i32, b: i32) -> Result<i32, DivisionError> {
    if b == 0 {
        return Err(DivisionError::DivideByZero);
    }
    Ok(a / b)
}

fn read_config(path: &str) -> Result<Config, ConfigError> {
    let content = std::fs::read_to_string(path).map_err(ConfigError::Io)?;
    toml::from_str(&content).map_err(ConfigError::Parse)
}
```

## Notes
- The `?` operator propagates `Err` (converting via `From` where needed) with the same one-line cost as `.unwrap()`, so there's no reason to reach for the panicking form in fallible code.
- `.map_err(...)` attaches context or converts one error type into another as it flows up through a call chain.
- `.ok()` discards the error and converts to `Option` — reach for it only when the caller genuinely doesn't care why something failed.
- Define error types with `thiserror` rather than hand-writing `Display`/`Error` boilerplate for every variant.

## References
- [err-thiserror-lib](err-thiserror-lib.md)
- [err-question-mark](err-question-mark.md)
- [type-option-nullable](type-option-nullable.md)
