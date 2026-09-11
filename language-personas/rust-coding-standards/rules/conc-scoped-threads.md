---
title: Use thread::scope to Borrow Stack Data Across Threads
impact: MEDIUM
impactDescription: Eliminates Arc/clone boilerplate for short-lived parallel tasks
tags: [concurrency, threads, scoped-threads, borrowing]
---

# Use thread::scope to Borrow Stack Data Across Threads [MEDIUM]

## Description
Scoped threads, stable since Rust 1.63, guarantee every thread spawned inside the scope joins before `thread::scope` returns. That lifetime guarantee lets spawned threads borrow non-`'static` data straight from the enclosing stack frame — no `Arc`, no cloning, no heap allocation required. For short parallel tasks that only need to read or write local data, scoped threads are simpler and cheaper than reaching for `Arc<Mutex<...>>`.

## Bad Example
```rust
use std::sync::Arc;
use std::thread;

fn parallel_sum(data: &[i64]) -> i64 {
    // Arc + clone just to share a slice -- heap allocation and boilerplate
    let data = Arc::new(data.to_vec());
    let mid = data.len() / 2;

    let data1 = Arc::clone(&data);
    let h1 = thread::spawn(move || data1[..mid].iter().sum::<i64>());

    let data2 = Arc::clone(&data);
    let h2 = thread::spawn(move || data2[mid..].iter().sum::<i64>());

    h1.join().unwrap() + h2.join().unwrap()
}
```

## Good Example
```rust
use std::thread;

fn parallel_sum(data: &[i64]) -> i64 {
    let mid = data.len() / 2;
    let (left, right) = data.split_at(mid);

    thread::scope(|s| {
        let h1 = s.spawn(|| left.iter().sum::<i64>());
        let h2 = s.spawn(|| right.iter().sum::<i64>());
        h1.join().unwrap() + h2.join().unwrap()
    })
}
```

## Notes
- The closure passed to `thread::scope` receives a `Scope<'env, '_>` handle; threads spawned via `s.spawn(...)` may borrow anything alive in `'env`.
- All threads are joined automatically when the scope returns, even if one of them panics — `thread::scope` itself then panics after joining the rest.
- For CPU-bound data-parallel work over a large homogeneous collection, prefer rayon's `par_iter()`, which handles chunking and work-stealing automatically.
- Reach for scoped threads when you need explicit control over a fixed number of distinct sub-tasks rather than a uniform collection.

## References
- [own-arc-shared](own-arc-shared.md)
- [conc-rayon-par-iter](conc-rayon-par-iter.md)
- [async-spawn-blocking](async-spawn-blocking.md)
