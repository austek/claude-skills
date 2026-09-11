---
title: Model-Check Concurrent Code with Loom
impact: HIGH
impactDescription: Exhaustively checks every interleaving the memory model permits, not just the ones your scheduler happened to hit
tags: [testing, concurrency, loom, atomics, correctness]
---

# Model-Check Concurrent Code with Loom [HIGH]

## Description
Hammering a lock-free structure with real OS threads a million times over only tells you that whatever interleaving your scheduler happened to pick on that run didn't trigger a bug — the specific ordering that would have broken things may simply never have occurred to the scheduler. `loom` sidesteps the scheduler entirely by substituting its own simulated versions of `AtomicBool`, `Mutex`, `thread::spawn`, and the rest, then enumerating every ordering and memory-reordering the C11 model permits for a given piece of code. A passing loom run is a claim about *all* legal interleavings within that model, not a sample of however many an OS scheduler happened to produce in a fixed amount of wall-clock time, which is the reason Tokio leans on it to check its own internal atomics and channels rather than trusting stress tests alone.

## Bad Example
```rust
// Passing a million iterations proves nothing about interleavings the
// scheduler never happened to try.
#[test]
fn stress_test_flag() {
    use std::sync::{atomic::{AtomicBool, Ordering}, Arc};
    let flag = Arc::new(AtomicBool::new(false));
    for _ in 0..1_000_000 {
        let flag = Arc::clone(&flag);
        std::thread::spawn(move || flag.store(true, Ordering::Relaxed));
    }
}
```

## Good Example
```rust
// src/flag.rs — swap in loom's instrumented types only when model-checking.
#[cfg(loom)]
use loom::sync::atomic::{AtomicBool, Ordering};
#[cfg(not(loom))]
use std::sync::atomic::{AtomicBool, Ordering};

pub struct Flag(AtomicBool);

impl Flag {
    pub fn set(&self) {
        self.0.store(true, Ordering::Release);
    }
    pub fn is_set(&self) -> bool {
        self.0.load(Ordering::Acquire)
    }
}

#[cfg(loom)]
mod tests {
    use super::Flag;
    use loom::sync::Arc;

    #[test]
    fn flag_set_visible_after_join() {
        loom::model(|| {
            let flag = Arc::new(Flag(loom::sync::atomic::AtomicBool::new(false)));
            let writer = {
                let flag = Arc::clone(&flag);
                loom::thread::spawn(move || flag.set())
            };
            writer.join().unwrap();
            assert!(flag.is_set());
        });
    }
}
```

## Notes
- Gate the swap behind `#[cfg(loom)]` / `#[cfg(not(loom))]` so production builds always link `std`'s real atomics, and only a dedicated `RUSTFLAGS="--cfg loom" cargo test` run pulls in loom's simulated versions.
- Keep each `loom::model` closure small — loom explores every legal schedule, and the number of schedules grows combinatorially with the number of threads and atomic operations in the closure, so a large model can take an impractically long time to finish.
- `loom::model(|| { ... })` is the entry point; loom repeatedly re-runs the closure under different orderings internally, so a single call to it already covers the exhaustive search.
- Loom only verifies conformance to the C11 memory model it simulates — it will not catch a purely logical bug that has nothing to do with thread interleaving, so keep ordinary unit tests for that.

## References
- [conc-atomic-ordering](../../rust-coding-standards/rules/conc-atomic-ordering.md)
- [test-criterion-bench](test-criterion-bench.md)
