---
title: Never Compare Floats With == — Use Tolerance or total_cmp
impact: HIGH
impactDescription: Eliminates rounding-driven false negatives and NaN-triggered sort panics
tags: [numeric, floating-point, comparison, nan]
---

# Never Compare Floats With == — Use Tolerance or total_cmp [HIGH]

## Description
Floating-point arithmetic is not exact: `0.1 + 0.2 == 0.3` evaluates to `false` in Rust, as in every IEEE 754 language, because neither value has an exact binary representation. Worse, `NaN != NaN` by the IEEE 754 standard, so any equality check involving `NaN` is always false, and `f64::partial_cmp` returns `None` on `NaN` — which makes a naive `sort_by(|a, b| a.partial_cmp(b).unwrap())` panic the moment a `NaN` appears in the data. Use an epsilon tolerance for approximate equality, and `f64::total_cmp` (stable since 1.62) for a total order that never panics.

## Bad Example
```rust
fn is_unit_length(x: f64, y: f64) -> bool {
    (x * x + y * y).sqrt() == 1.0 // almost always false due to rounding
}

fn sort_scores(scores: &mut Vec<f64>) {
    scores.sort_by(|a, b| a.partial_cmp(b).unwrap()); // panics if any score is NaN
}
```

## Good Example
```rust
fn approx_eq(a: f64, b: f64, epsilon: f64) -> bool {
    (a - b).abs() < epsilon
}

fn is_unit_length(x: f64, y: f64) -> bool {
    approx_eq((x * x + y * y).sqrt(), 1.0, 1e-9)
}

// total_cmp defines a strict order over all floats, including NaN -- never panics
fn sort_scores(scores: &mut Vec<f64>) {
    scores.sort_by(|a, b| a.total_cmp(b));
}
```

## Notes
- An absolute epsilon (`1e-9`) is simple but wrong for very large or very small magnitudes; a relative epsilon (`(a-b).abs() / a.abs().max(b.abs()) < epsilon`) is more robust but needs a zero-case guard.
- `f64::total_cmp` orders `-NaN < -∞ < … < -0.0 < +0.0 < … < +∞ < NaN` and is available on both `f32` and `f64`.
- Check `is_nan()` / `is_finite()` before arithmetic on untrusted floating-point input.
- `==` on floats is still valid for bit-exact checks (e.g., "did this value change since last write") — document that intent explicitly when you use it.

## References
- [num-overflow-explicit](num-overflow-explicit.md)
