---
title: Structure Branches to Hint the Likely Path
impact: LOW
impactDescription: Reduces branch mispredictions, which cost 10-20 stalled cycles each
tags: [optimization, branch-prediction, control-flow, hints]
---

# Structure Branches to Hint the Likely Path [LOW]

## Description
Modern CPUs speculatively execute past a branch based on prediction; a misprediction flushes the pipeline and costs roughly 10-20 cycles. On stable Rust, code structure itself is the primary hint: an early `return` for the rare case, extracting the rare branch into its own (often `#[cold]`) function, and ordering `if`/`else` so the common case comes first all bias the compiler's layout and prediction toward treating the fall-through path as hot. Nightly offers explicit `std::hint::likely`/`unlikely`, and stable Rust 1.95+ has `std::hint::cold_path()` for marking a branch as rare without restructuring.

## Bad Example
```rust
fn process(data: Option<&Data>) -> i32 {
    match data {
        Some(d) => complex_processing(d),
        None => 0, // no hint that this is the rare case
    }
}
```

## Good Example
```rust
fn process(data: Option<&Data>) -> i32 {
    // Early return for the unlikely case biases the compiler toward
    // treating the code after it as the hot, fall-through path.
    let data = match data {
        None => return 0,
        Some(d) => d,
    };
    complex_processing(data)
}
```

## Notes
- Match arm ordering matters too: list the most common variant first when the match compiles to a chain of comparisons.
- `std::hint::cold_path()` (stable since 1.95) marks the enclosing branch as unlikely — call it inside the rare arm rather than restructuring control flow.
- Nightly's `std::hint::likely`/`unlikely` (feature `likely_unlikely`) give explicit control but are unstable as of this writing.
- Don't guess which branch is likely — profile first (`perf record`, `cargo flamegraph`); a wrong hint pessimizes the actually-common path.

## References
- [opt-cold-unlikely](opt-cold-unlikely.md)
- [opt-inline-never-cold](opt-inline-never-cold.md)
