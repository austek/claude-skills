---
title: Configure the Tokio Runtime for the Actual Workload
impact: MEDIUM
impactDescription: Avoids wasted threads on CPU-bound work and thread starvation on IO-bound work
tags: [async, runtime, tokio, configuration]
---

# Configure the Tokio Runtime for the Actual Workload [MEDIUM]

## Description
`#[tokio::main]`'s default multi-threaded runtime is tuned for the common case of many concurrent I/O-bound tasks, but it isn't universally correct. CPU-heavy work awaited directly on that runtime still starves the same limited worker pool that `spawn_blocking` exists to protect against, and a single-connection CLI client gains nothing from multiple worker threads while paying their scheduling overhead. Tokio exposes the knobs to match the runtime to the workload: `current_thread` for single-connection or test scenarios, an explicit `worker_threads` count for tuned multi-threaded I/O work, and a manually built `Builder` when a program genuinely needs separate runtimes for separate kinds of work.

## Bad Example
```rust
// Default multi-threaded runtime used unconditionally for a
// single-connection client — extra worker threads add scheduling
// overhead this workload never benefits from.
#[tokio::main]
async fn main() {
    let client = Client::new();
    client.run().await;
}
```

## Good Example
```rust
// current_thread fits a single-connection, single-task workload —
// simpler to reason about and to debug than a multi-worker scheduler.
#[tokio::main(flavor = "current_thread")]
async fn main() {
    let client = Client::new();
    client.run().await;
}

// Many concurrent connections: keep the default multi-threaded runtime,
// and size worker_threads deliberately rather than accepting whatever
// the host's core count happens to be.
#[tokio::main(worker_threads = 4)]
async fn main() {
    // ...
}
```

## Notes
- `std::thread::available_parallelism()` (stable since 1.59) is the right source for core count when sizing a `Builder` manually — unlike the unmaintained `num_cpus` crate, it respects cgroup CPU quotas in containerized deployments.
- IO-bound workloads can benefit from `worker_threads` oversubscribed beyond the core count, since most of each task's time is spent waiting rather than computing; CPU-bound workloads gain nothing past the core count and should match it exactly.
- `#[tokio::test(flavor = "multi_thread", worker_threads = n)]` is the way to exercise genuine cross-thread races in a test; the default single-threaded test runtime silently hides bugs that only surface under real concurrency.
- Running two `Builder`-constructed runtimes side by side (one tuned for I/O, one for CPU work via its own `spawn_blocking` pool) is a valid pattern for a program with genuinely bimodal workloads, at the cost of managing two schedulers explicitly.
- `max_blocking_threads` on a `Builder` caps the pool that `spawn_blocking` and `tokio::fs` draw from — it defaults to 512, but a program spawning enough blocking work to approach that limit needs this tuned deliberately, not left implicit.

## References
- [async-spawn-blocking](async-spawn-blocking.md)
- [async-no-lock-await](async-no-lock-await.md)
- [async-joinset-structured](async-joinset-structured.md)
