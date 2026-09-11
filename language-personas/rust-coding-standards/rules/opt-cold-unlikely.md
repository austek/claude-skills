---
title: Mark Rarely-Taken Paths With #[cold]
impact: LOW
impactDescription: Keeps rare code out of the hot instruction-cache footprint
tags: [optimization, cold, branch-prediction, code-layout]
---

# Mark Rarely-Taken Paths With #[cold] [LOW]

## Description
`#[cold]` tells the compiler a function is rarely called, which changes three things: the function is placed in a separate code section away from hot code (better instruction-cache utilization), the compiler emits branch hints favoring the non-cold path, and the cold function is deprioritized for inlining and optimization effort. Extracting rare branches — error construction, panics, logging of unusual events — into small `#[cold]` functions keeps the hot path itself compact and easy for the branch predictor to get right.

## Bad Example
```rust
fn validate(input: &str) -> Result<Data, ValidationError> {
    if input.is_empty() {
        return Err(ValidationError::Empty); // error construction inlined into hot path
    }
    if input.len() > 1000 {
        return Err(ValidationError::TooLong);
    }
    Ok(parse_data(input))
}
```

## Good Example
```rust
fn validate(input: &str) -> Result<Data, ValidationError> {
    if input.is_empty() {
        return cold_empty_error();
    }
    if input.len() > 1000 {
        return cold_too_long_error();
    }
    Ok(parse_data(input))
}

#[cold]
fn cold_empty_error() -> Result<Data, ValidationError> {
    Err(ValidationError::Empty)
}

#[cold]
fn cold_too_long_error() -> Result<Data, ValidationError> {
    Err(ValidationError::TooLong)
}
```

## Notes
- Combine with `#[inline(never)]` for the strongest effect — `#[cold]` alone doesn't prevent inlining, it only deprioritizes it.
- Good candidates: error construction, panic paths, rare-event logging, and fallback code that should almost never execute.
- Only apply this to code proven cold by profiling — misjudging a path as cold pessimizes it for the common case.
- Verify placement with `objdump` (look for `.cold` sections) or `nm | grep cold` rather than assuming the attribute took effect.

## References
- [opt-inline-never-cold](opt-inline-never-cold.md)
- [opt-likely-hint](opt-likely-hint.md)
- [err-result-over-panic](err-result-over-panic.md)
