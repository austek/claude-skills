---
title: Avoid as for Narrowing Casts — Use TryFrom
impact: HIGH
impactDescription: Prevents silent truncation bugs that as casts hide from review
tags: [numeric, casting, try-from, type-conversion]
---

# Avoid as for Narrowing Casts — Use TryFrom [HIGH]

## Description
The `as` operator silently truncates on a narrowing cast — `300u32 as u8` is `44`, with no warning at compile time or panic at runtime. Float-to-integer casts add their own surprises: out-of-range values saturate to the target's min/max (since Rust 1.45), but `NaN` becomes `0`. These failure modes are easy to miss in review and won't surface without a test that happens to hit the boundary. `From`/`Into` only compile when a conversion is provably lossless; `TryFrom`/`TryInto` return a `Result`, forcing the fallible case into the type signature where a reviewer and the compiler both see it.

## Bad Example
```rust
fn narrow(x: u32) -> u8 {
    x as u8 // silently truncates: 300 becomes 44
}

fn to_index(f: f64) -> usize {
    f as usize // NaN becomes 0, negatives become 0, may truncate
}
```

## Good Example
```rust
use std::convert::TryFrom;

// Widening: From<u8> for u32 is always lossless -- won't compile if lossy.
fn widen(x: u8) -> u32 {
    u32::from(x)
}

// Narrowing: TryFrom makes the failure case explicit and typed.
fn narrow(x: u32) -> Result<u8, <u8 as TryFrom<u32>>::Error> {
    u8::try_from(x)
}
```

## Notes
- `From<A> for B` compiles only when the conversion is always lossless — attempting `u8::from(300u32)` is a compile error, not a runtime surprise.
- `TryFrom` returns `Result<T, TryFromIntError>` from the standard library with no extra crates needed.
- Reserve `as` for intentional pointer casts, float-to-integer conversions where the range has already been validated and documented, and `usize` ↔ pointer-sized-integer conversions.
- `.try_into()` often needs a type annotation or turbofish to help inference: `let n: u8 = x.try_into()?;`

## References
- [conv-tryfrom-fallible](conv-tryfrom-fallible.md)
- [num-overflow-explicit](num-overflow-explicit.md)
