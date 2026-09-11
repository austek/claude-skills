---
title: Use Rayon's par_iter for CPU-Bound Data Parallelism
impact: HIGH
impactDescription: Near-linear speedup across cores from a near one-word iterator change
tags: [concurrency, rayon, parallelism, iterators]
---

# Use Rayon's par_iter for CPU-Bound Data Parallelism [HIGH]

## Description
Rayon's work-stealing scheduler parallelizes data-parallel workloads through an API nearly identical to the standard iterator traits: swapping `.iter()` for `.par_iter()` is often the entire change needed for near-linear speedup across cores. It balances load automatically, handles chunking, and composes with the full iterator adapter chain, so existing map/filter/fold pipelines port over directly. Rayon is strictly for CPU-bound computation — IO-bound concurrency belongs to an async runtime instead, since rayon's threads block.

## Bad Example
```rust
// Single-threaded -- leaves every other core idle on a CPU-bound workload
fn sum_squares(data: &[f64]) -> f64 {
    data.iter().map(|x| x * x).sum()
}

fn normalize(data: &mut [f64]) {
    let max = data.iter().cloned().fold(f64::NEG_INFINITY, f64::max);
    data.iter_mut().for_each(|x| *x /= max);
}
```

## Good Example
```rust
use rayon::prelude::*;

fn sum_squares(data: &[f64]) -> f64 {
    data.par_iter().map(|x| x * x).sum()
}

fn normalize(data: &mut [f64]) {
    let max = data.par_iter().cloned().reduce(|| f64::NEG_INFINITY, f64::max);
    data.par_iter_mut().for_each(|x| *x /= max);
}

fn sort_large(data: &mut [f64]) {
    data.par_sort_unstable_by(|a, b| a.partial_cmp(b).unwrap());
}
```

## Notes
- `use rayon::prelude::*;` enables `.par_iter()` on slices and most standard collections.
- For small collections, sequential often wins — thread-spawn overhead dominates when per-element work is trivial; profile before switching.
- `with_min_len()` / `with_max_len()` tune the chunk granularity rayon uses when splitting work.
- Rayon does not prevent data races on shared state reached from inside the closure — still guard it with a `Mutex` or atomics.

## References
- [conc-scoped-threads](conc-scoped-threads.md)
- [async-spawn-blocking](async-spawn-blocking.md)
- [Rayon documentation](https://docs.rs/rayon)
