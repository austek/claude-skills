---
title: Notice a for i in 0..len Loop Where an Iterator Was Available
impact: MEDIUM
impactDescription: A common porting habit that reintroduces bounds checks and off-by-one risk
tags: [anti-pattern, iterators, bounds-checking, code-smell]
---

# Notice a for i in 0..len Loop Where an Iterator Was Available [MEDIUM]

## Description
`for i in 0..data.len() { ... data[i] ... }` is the loop shape a C, Java, or Python programmer reaches for by habit, and it compiles in Rust without complaint — which is exactly why it shows up in code written by someone porting logic from another language, or under time pressure, even though `data.iter()` almost always expresses the same intent more safely and with less code. The index variable adds two costs the iterator form doesn't have: a bounds check the compiler has to insert at every access because it can't always prove `i` stays in range, and an off-by-one risk (`<=` instead of `<`, an unchecked `i + 1`) that simply doesn't exist when there's no manually managed index at all.

## Bad Example
```rust
fn normalize(data: &mut [f64]) {
    let max = data.iter().cloned().fold(0.0, f64::max);
    for i in 0..data.len() {
        data[i] /= max; // bounds-checked, and a stray `<=` here would be a live bug
    }
}
```

## Good Example
```rust
fn normalize(data: &mut [f64]) {
    let max = data.iter().cloned().fold(0.0, f64::max);
    for x in data.iter_mut() {
        *x /= max; // no index to get wrong, no bounds check to eliminate
    }
}
```

## Notes
- The conversion is usually mechanical: `for i in 0..v.len() { v[i] }` becomes `for x in &v`, and the mutable form becomes `for x in v.iter_mut() { *x = ... }` — reach for `.enumerate()` on the rare occasion the index itself is genuinely part of the output.
- A real exception is non-sequential access — swapping paired elements (`data.swap(i, mid + i)`) or walking two dimensions of a matrix — where an iterator form would be more convoluted than the indexed version, not less.
- This is the same underlying issue `perf-iter-over-index` covers from the performance angle — this entry is about recognizing the pattern during review or when reading unfamiliar code, not a separate technique.

## References
- [perf-iter-over-index](perf-iter-over-index.md)
- [opt-bounds-check](opt-bounds-check.md)
- [perf-iter-lazy](perf-iter-lazy.md)
