---
title: Use spawn_blocking for CPU-Intensive or Blocking Work
impact: CRITICAL
impactDescription: Prevents one heavy call from starving every other task on the runtime
tags: [async, spawn-blocking, tokio, cpu-bound]
---

# Use spawn_blocking for CPU-Intensive or Blocking Work [CRITICAL]

## Description
Tokio's async worker threads are a small, fixed pool cooperatively shared by every task scheduled on the runtime. Any call that doesn't yield — a tight CPU-bound computation, a synchronous filesystem read, a blocking mutex from `std` held indefinitely, `std::thread::sleep` — occupies its worker thread for the call's entire duration, and every other task assigned to that thread simply doesn't run until it's done. `tokio::task::spawn_blocking` moves the closure onto a separate, uncapped blocking-thread pool built for exactly this, returning a `JoinHandle` the async side can await without itself blocking. The rule of thumb: anything that isn't `.await`-based and takes more than roughly a millisecond belongs in `spawn_blocking`, not directly in async code.

## Bad Example
```rust
// CPU-intensive work runs directly on an async worker thread — every
// other task scheduled on that thread stalls until this returns.
async fn process_image(data: &[u8]) -> ProcessedImage {
    let resized = resize_image(data);
    compress(resized)
}
```

## Good Example
```rust
use tokio::task;

async fn process_image(data: Vec<u8>) -> ProcessedImage {
    task::spawn_blocking(move || {
        let resized = resize_image(&data);
        compress(resized)
    })
    .await
    .expect("blocking task panicked")
}
```

## Notes
- `spawn_blocking`'s `JoinHandle` resolves to a `Result` whose `Err` means the closure panicked — propagate or log it explicitly rather than unwrapping blindly, since a panic there doesn't otherwise surface as a runtime crash.
- Synchronous file I/O specifically has a purpose-built alternative — `tokio::fs`, which wraps `spawn_blocking` internally — so reach for it directly rather than hand-rolling the same wrapper around `std::fs`.
- CPU-parallel work (`rayon`) composes with this rule by running `par_iter()` inside the `spawn_blocking` closure, keeping the parallel computation off the async worker pool entirely.
- The blocking-thread pool is much larger than the async worker pool but still finite (`Builder::max_blocking_threads`, default 512) — spawning enough blocking tasks to exhaust it delays new ones from starting, so it isn't a license for unlimited concurrency either.
- A `std::sync::Mutex` guard held only within a synchronous, non-awaiting scope is fine on an async thread — the problem this rule addresses is unyielding work occupying the thread, not synchronization itself.

## References
- [async-tokio-fs](async-tokio-fs.md)
- [async-no-lock-await](async-no-lock-await.md)
- [async-tokio-runtime](async-tokio-runtime.md)
