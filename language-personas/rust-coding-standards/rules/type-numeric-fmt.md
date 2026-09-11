---
title: Implement Hex/Octal/Binary Formatting for Numeric Newtypes
impact: LOW
impactDescription: Keeps {:x}/{:o}/{:b} working on a newtype instead of a compile error
tags: [type-safety, formatting, newtype, api-guidelines]
---

# Implement Hex/Octal/Binary Formatting for Numeric Newtypes [LOW]

## Description
The Rust API Guidelines (C-NUM-FMT) expect a numeric type to support `{:x}`, `{:X}`, `{:o}`, and `{:b}` wherever its underlying integer does. A numeric newtype that only implements `Display` silently drops that expectation: a caller who reaches for `{:x}` to inspect a bitmask or address hits a compile error instead of the formatted output they expect from any other integer-like type. Each trait — `LowerHex`, `UpperHex`, `Octal`, `Binary` — is a one-line forward to the inner value's own implementation, so there's no real cost to providing them.

## Bad Example
```rust
struct Mask(u32);

impl std::fmt::Display for Mask {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        write!(f, "{}", self.0)
    }
}
// println!("{:x}", Mask(0xDEAD_BEEF)); // compile error: no LowerHex impl
```

## Good Example
```rust
use std::fmt;

struct Mask(u32);

impl fmt::LowerHex for Mask {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        fmt::LowerHex::fmt(&self.0, f) // forwards flags like # and width too
    }
}

impl fmt::UpperHex for Mask {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        fmt::UpperHex::fmt(&self.0, f)
    }
}
```

## Notes
- Forward through the inner type's own trait `fmt` implementation so format flags (`#`, `0`-padding, width) are handled correctly without reimplementing them.
- Apply this to any newtype wrapping a primitive integer (`u8`–`u128`, `i8`–`i128`, `usize`, `isize`).
- Skip `Octal`/`Binary` when there's no domain reason to print in those bases (a purely decimal `Count`), but implement `LowerHex`/`UpperHex` for any mask, address, or identifier-flavored type.
- `#[derive(...)]` doesn't cover these traits — each one needs an explicit (if trivial) `impl`.

## References
- [type-newtype-ids](type-newtype-ids.md)
- [type-display-vs-debug](type-display-vs-debug.md)
