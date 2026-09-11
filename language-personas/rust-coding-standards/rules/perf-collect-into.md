---
title: Reuse a Collection's Allocation Instead of Collecting Fresh Each Time
impact: LOW
impactDescription: Removes one allocation per loop iteration in a repeated collect pattern
tags: [performance, iterators, allocation, collections]
---

# Reuse a Collection's Allocation Instead of Collecting Fresh Each Time [LOW]

## Description
`.collect::<Vec<_>>()` inside a loop body allocates a brand new `Vec` every pass, even when the previous pass's buffer is sitting right there unused after the loop moves on. `Iterator::collect_into` would be the direct fix — clear the buffer, collect into it, repeat — but as of this writing it's still gated behind the nightly-only `iter_collect_into` feature (tracking issue `#94780`), not something stable code can reach for. `Extend::extend` is the stable equivalent: clearing a `Vec` drops its elements but keeps the allocated capacity, and `extend` fills that same buffer from an iterator without asking the allocator for anything new.

## Bad Example
```rust
fn filter_batches(data: &[Vec<i32>]) {
    for batch in data {
        // A fresh Vec, and a fresh allocation, every single iteration.
        let filtered: Vec<_> = batch.iter().filter(|&&x| x > 0).copied().collect();
        process(&filtered);
    }
}
```

## Good Example
```rust
fn filter_batches(data: &[Vec<i32>]) {
    let mut buffer = Vec::new();
    for batch in data {
        buffer.clear(); // drops elements, keeps the allocation
        buffer.extend(batch.iter().filter(|&&x| x > 0).copied());
        process(&buffer);
    }
}
```

## Notes
- `collect_into` isn't available on stable Rust — reach for `extend` on stable, and only consider `collect_into` behind `#![feature(iter_collect_into)]` in nightly-only benchmarking or experimentation code.
- This pattern applies to any type implementing `Extend`, not just `Vec` — `HashSet`, `HashMap`, and `VecDeque` all support `.clear()` + `.extend()` the same way.
- The buffer needs to actually outlive the loop iteration for this to pay off — a buffer declared fresh inside the loop body gets a new allocation on every pass regardless of whether `collect` or `extend` is used to fill it.

## References
- [perf-drain-reuse](perf-drain-reuse.md)
- [mem-reuse-collections](mem-reuse-collections.md)
- [perf-extend-batch](perf-extend-batch.md)
