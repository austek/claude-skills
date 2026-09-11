---
title: Bound Values With clamp and Saturating Arithmetic
impact: LOW
impactDescription: Replaces error-prone hand-rolled min/max chains with a single, correct call
tags: [numeric, clamp, saturating, bounds]
---

# Bound Values With clamp and Saturating Arithmetic [LOW]

## Description
Constraining a value to a range is common, but a hand-rolled `if`/`min`/`max` chain is verbose and easy to get subtly wrong, especially when signed and unsigned types mix. `Ord::clamp(min, max)` expresses the intent in one call and is available on every `Ord` type, including all integer primitives; `f32`/`f64` have their own `clamp` since Rust 1.50. When the goal is arithmetic that stops at a type's own limits rather than panicking or wrapping, combine `clamp` with the `saturating_*` methods instead of writing the bound check by hand.

## Bad Example
```rust
fn apply_damage(health: i32, damage: i32) -> i32 {
    let result = health - damage;
    if result < 0 { 0 } else { result } // verbose, easy to mis-order
}
```

## Good Example
```rust
fn apply_damage(health: i32, damage: i32) -> i32 {
    // saturating_sub stops at i32::MIN, then clamp enforces non-negative
    health.saturating_sub(damage).clamp(0, i32::MAX)
}

fn normalize_alpha(a: f32) -> f32 {
    a.clamp(0.0, 1.0) // NaN propagates: NaN.clamp(..) == NaN
}
```

## Notes
- `clamp` panics if `min > max` — validate ordering first when bounds come from user input.
- `f32`/`f64` `clamp` propagates `NaN` when `self` is `NaN`; the result is unspecified if `min` or `max` is `NaN` — filter `NaN` before calling it on untrusted floats.
- `saturating_*` bounds a single arithmetic operation at the type's absolute limits; `clamp` bounds the final value to arbitrary limits — they compose, as in `apply_damage`.
- Prefer `clamp` over nested `min`/`max` calls purely for readability even when both are otherwise correct.

## References
- [num-overflow-explicit](num-overflow-explicit.md)
- [num-cast-try-from](num-cast-try-from.md)
