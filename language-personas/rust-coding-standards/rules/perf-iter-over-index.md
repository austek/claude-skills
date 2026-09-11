---
title: Traverse Slices with Iterators, Not Manual Index Loops
impact: MEDIUM
impactDescription: Removes per-access bounds checks and opens the door to auto-vectorization
tags: [performance, iterators, bounds-checking, simd]
---

# Traverse Slices with Iterators, Not Manual Index Loops [MEDIUM]

## Description
`for i in 0..data.len() { data[i] }` asks the compiler to prove, at every single access, that `i` is still in bounds — and because the index comes from an arbitrary counter the compiler often can't discharge that proof once and for all, so the bounds check stays in the generated code for every iteration. `data.iter()` sidesteps the problem structurally: the iterator itself only ever produces valid positions within the slice, so there's no separate index to validate against a bound on each step. That difference matters beyond just skipping a branch — with bounds checks out of the way, LLVM has a much easier time recognizing the loop as vectorizable (auto-SIMD), something an index-based loop with implicit bounds checks frequently blocks.

## Bad Example
```rust
fn dot_product(a: &[f64], b: &[f64]) -> f64 {
    let mut sum = 0.0;
    for i in 0..a.len().min(b.len()) {
        sum += a[i] * b[i]; // bounds-checked against both slices, every iteration
    }
    sum
}
```

## Good Example
```rust
fn dot_product(a: &[f64], b: &[f64]) -> f64 {
    a.iter().zip(b.iter()).map(|(&x, &y)| x * y).sum() // no index, no bounds check
}

fn double_in_place(data: &mut [i32]) {
    for x in data.iter_mut() {
        *x *= 2;
    }
}
```

## Notes
- Indices are still the right tool when the access pattern is genuinely non-sequential — `data.swap(i, j)` between two computed positions, for instance, has no natural iterator form.
- `.enumerate()` gets the index alongside the value without reintroducing manual bounds-checked indexing — reach for it whenever the position itself (not just the value) is needed inside the loop body.
- `rayon`'s `.par_iter()` extends this same iterator style to data-parallel execution across threads — a codebase already using iterators over indices is a much shorter step away from parallelizing a hot loop than one built on manual index arithmetic.

## References
- [perf-iter-lazy](perf-iter-lazy.md)
- [opt-bounds-check](opt-bounds-check.md)
- [anti-index-over-iter](anti-index-over-iter.md)
- [conc-rayon-par-iter](conc-rayon-par-iter.md)
