---
title: Use NonZero Types to Forbid Zero at the Type Level
impact: MEDIUM
impactDescription: Removes defensive zero-checks and gives Option<NonZero> free niche layout
tags: [numeric, nonzero, niche-optimization, type-safety]
---

# Use NonZero Types to Forbid Zero at the Type Level [MEDIUM]

## Description
`NonZeroU32`, `NonZeroI64`, and their siblings in `std::num` make zero unrepresentable at the type level: the only way to construct one is `NonZeroU32::new(n)`, which returns `Option<NonZeroU32>`. That pushes the zero-check to the single construction site instead of scattering defensive `assert!(x != 0)` checks through every function that uses the value. As a bonus, the compiler uses the impossible zero bit-pattern as a *niche*, so `Option<NonZeroU32>` is exactly the same size as a bare `u32` — the `Option` tag costs nothing.

## Bad Example
```rust
// Caller must remember never to pass 0 -- nothing enforces it.
fn divide(numerator: u32, denominator: u32) -> u32 {
    assert!(denominator != 0, "denominator must not be zero");
    numerator / denominator
}
```

## Good Example
```rust
use std::num::NonZeroU32;

// Zero is rejected at construction; the division itself is always safe.
fn divide(numerator: u32, denominator: NonZeroU32) -> u32 {
    numerator / denominator.get()
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
struct WidgetId(NonZeroU32);

impl WidgetId {
    fn new(id: u32) -> Option<Self> {
        NonZeroU32::new(id).map(WidgetId)
    }
}
```

## Notes
- All primitive widths are covered: `NonZeroU8` through `NonZeroU128`/`NonZeroUsize`, and the signed counterparts.
- `.get()` returns the inner primitive; `NonZeroU32` doesn't implement `Add`/`Sub` directly since the result could be zero — extract, compute, and reconstruct with `NonZeroU32::new(result)?`.
- The niche optimization extends to custom newtypes wrapping a `NonZero*` field, not just the standard-library types directly.
- Reserve `.expect("n must be non-zero")` for well-verified program boundaries (e.g., a hardcoded constant), not for fallible production input paths.

## References
- [type-newtype-ids](type-newtype-ids.md)
- [mem-smaller-integers](mem-smaller-integers.md)
