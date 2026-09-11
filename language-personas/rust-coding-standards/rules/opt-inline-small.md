---
title: Use #[inline] for Small Cross-Crate Hot Functions
impact: LOW
impactDescription: Removes call overhead that can dominate a small function's own cost
tags: [optimization, inlining, inline-hint, cross-crate]
---

# Use #[inline] for Small Cross-Crate Hot Functions [LOW]

## Description
Function-call overhead — stack frame setup, register saves, the jump itself — can outweigh the work done by a genuinely small function. The compiler often inlines automatically within a single crate, but without Link-Time Optimization it cannot inline a function across a crate boundary unless that function's body was made available via `#[inline]`. Marking small, frequently-called public functions with `#[inline]` lets downstream crates inline them instead of paying a call for a one-line body.

## Bad Example
```rust
// No inline hint: may stay a real call across crate boundaries
pub fn is_ascii_digit(b: u8) -> bool {
    b >= b'0' && b <= b'9'
}
```

## Good Example
```rust
#[inline]
pub fn is_ascii_digit(b: u8) -> bool {
    b >= b'0' && b <= b'9'
}
```

## Notes
- Without `#[inline]` (or LTO), a downstream crate calling `is_ascii_digit` pays a real function call even though the body is one comparison.
- Reserve `#[inline(always)]` for functions a benchmark proved must be forced — `#[inline]` is a hint the compiler can still decline for large or rarely-hot functions.
- `#[inline(never)]` is the opposite tool: use it for large or cold functions where inlining would only bloat call sites.
- Enabling LTO (`opt-lto-release`) makes cross-crate inlining possible even without `#[inline]`, but the attribute still documents intent and helps without LTO enabled.

## References
- [opt-inline-always-rare](opt-inline-always-rare.md)
- [opt-inline-never-cold](opt-inline-never-cold.md)
- [opt-cold-unlikely](opt-cold-unlikely.md)
- [opt-lto-release](opt-lto-release.md)
