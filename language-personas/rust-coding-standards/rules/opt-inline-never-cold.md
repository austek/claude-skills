---
title: Extract Error and Cold Paths With #[inline(never)]
impact: LOW
impactDescription: Keeps error-construction code out of the hot path's instruction cache
tags: [optimization, inlining, inline-never, error-handling]
---

# Extract Error and Cold Paths With #[inline(never)] [LOW]

## Description
Inlining error-construction code directly into a hot function pollutes it with instructions that only ever run on the failure path, wasting instruction-cache space and blocking other optimizations the compiler could otherwise make on the hot code. Pulling that construction into a separate function marked `#[inline(never)]` — usually paired with `#[cold]` — keeps the common path small and lets the compiler apply better branch-prediction hints and code layout around the call.

## Bad Example
```rust
fn process_data(data: &[u8]) -> Result<Output, Error> {
    if data.is_empty() {
        // Error construction inlined directly into the hot function
        return Err(Error::Empty {
            context: format!("expected data, got empty slice"),
            suggestions: vec!["check input", "validate before calling"],
        });
    }
    do_processing(data)
}
```

## Good Example
```rust
fn process_data(data: &[u8]) -> Result<Output, Error> {
    if data.is_empty() {
        return Err(empty_data_error()); // hot path stays small
    }
    do_processing(data)
}

#[cold]
#[inline(never)]
fn empty_data_error() -> Error {
    Error::Empty {
        context: format!("expected data, got empty slice"),
        suggestions: vec!["check input", "validate before calling"],
    }
}
```

## Notes
- Pair `#[inline(never)]` with `#[cold]` — the two attributes address different concerns (inlining decision vs. code placement and branch hints) and compound.
- Good targets: panic paths (`fn ... -> !`), verbose error construction, and rarely-executed fallback branches.
- The stable `std::hint::cold_path()` (Rust 1.95+) marks a branch as unlikely without extracting a function, useful when extraction would be awkward.
- Structuring the hot path as the fall-through case (early `return` for the rare branch) reinforces the same hint the compiler already infers from control flow.

## References
- [opt-inline-small](opt-inline-small.md)
- [opt-inline-always-rare](opt-inline-always-rare.md)
- [err-result-over-panic](err-result-over-panic.md)
