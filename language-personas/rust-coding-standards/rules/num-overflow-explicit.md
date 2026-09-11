---
title: Handle Integer Overflow Explicitly
impact: CRITICAL
impactDescription: Eliminates the panic-in-debug/wrap-in-release split that hides real bugs
tags: [numeric, overflow, checked-arithmetic, safety]
---

# Handle Integer Overflow Explicitly [CRITICAL]

## Description
Integer overflow panics in debug builds but silently wraps (two's complement) in release builds by default. Relying on either behavior is a latent bug: the debug build masks it with a panic that never ships, and the release build produces a wrong result with no diagnostic at all. Choosing one of the explicit overflow-handling method families — `checked_*`, `saturating_*`, `wrapping_*`, `overflowing_*` — makes the intended behavior unmistakable to both the compiler and the next reader, and makes it identical across build profiles.

## Bad Example
```rust
fn add_score(current: u32, delta: u32) -> u32 {
    current + delta // panics in debug, wraps silently in release
}
```

## Good Example
```rust
// checked_add: returns None on overflow -- propagate or handle the error
fn add_score(current: u32, delta: u32) -> Option<u32> {
    current.checked_add(delta)
}

// saturating_add: clamps at the type's bound, no error path needed
fn increment_saturating(c: u8) -> u8 {
    c.saturating_add(1)
}
```

## Notes
- `checked_*` returns `Option<T>` — use when overflow is an error the caller must handle.
- `saturating_*` returns `T` clamped at the type's bounds — use when clamping is itself the correct behavior.
- `wrapping_*` returns `T` with modular arithmetic — use only when wraparound is intentional (checksums, ring buffer indices).
- `overflowing_*` returns `(T, bool)` — use when both the result and the overflow flag are needed, e.g. for carry propagation.
- All four families exist for every primitive integer type and every arithmetic operation (`add`, `sub`, `mul`, `div`, `shl`, `shr`).

## References
- [num-saturating-clamp](num-saturating-clamp.md)
- [num-cast-try-from](num-cast-try-from.md)
