---
title: Prefer Iterator Patterns That Eliminate Bounds Checks
impact: MEDIUM
impactDescription: Removes per-access bounds checks and unlocks vectorization in hot loops
tags: [optimization, performance, iterators, bounds-checking]
---

# Prefer Iterator Patterns That Eliminate Bounds Checks [MEDIUM]

## Description
Rust's safety guarantees require a bounds check on every slice or array index. In a tight loop that overhead adds up — both the branch itself and the way it blocks the compiler from vectorizing the loop. Iterator adaptors like `zip`, `windows`, and `chunks_exact` give the compiler enough information to prove indices are always in range, so it emits direct memory access with no check at all — without giving up memory safety. Elimination isn't guaranteed; verify hot code with generated assembly before relying on it.

## Bad Example
```rust
fn sum_products(a: &[f64], b: &[f64]) -> f64 {
    let mut sum = 0.0;
    for i in 0..a.len() {
        sum += a[i] * b[i]; // two bounds checks per iteration
    }
    sum
}
```

## Good Example
```rust
fn sum_products(a: &[f64], b: &[f64]) -> f64 {
    // zip proves both indices are in range once, up front
    a.iter().zip(b.iter()).map(|(x, y)| x * y).sum()
}

fn apply_filter(data: &[u8]) -> Vec<u8> {
    data.windows(3)
        .map(|w| ((w[0] as u16 + w[1] as u16 + w[2] as u16) / 3) as u8)
        .collect()
}
```

## Notes
- `get_unchecked` / `get_unchecked_mut` remove the check explicitly, but only after the caller proves the bounds with an `assert!` — document the invariant with a `SAFETY` comment.
- Random-access patterns (lookup by arbitrary index) can't avoid the check; use `.get()` and accept it rather than reaching for `unsafe`.
- Verify elimination actually happened with `cargo asm` or similar — the compiler's heuristics aren't guaranteed, so measure before trusting a pattern in a hot path.
- Don't apply this outside measured hot paths; readability loses to an unmeasured micro-optimization.

## References
- [opt-cache-friendly](opt-cache-friendly.md)
- [opt-simd-portable](opt-simd-portable.md)
