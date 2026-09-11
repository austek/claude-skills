---
title: Use the Weakest Correct Memory Ordering for Atomics
impact: HIGH
impactDescription: Avoids full memory barriers on every atomic op while staying data-race-free
tags: [concurrency, atomics, memory-ordering, lock-free]
---

# Use the Weakest Correct Memory Ordering for Atomics [HIGH]

## Description
Defaulting every atomic operation to `Ordering::SeqCst` is a correctness-first shortcut with a real cost: on ARM and RISC-V, `SeqCst` compiles to full memory barriers, while weaker orderings map to cheaper instructions. The bigger risk runs the other way — picking an ordering that's too weak is a correctness bug the compiler cannot catch, since it silently permits a data race instead of failing to compile. Choosing the right level requires understanding what each one actually guarantees, not defaulting to the strongest one out of caution.

## Bad Example
```rust
use std::sync::atomic::{AtomicBool, AtomicU64, Ordering};

static READY: AtomicBool = AtomicBool::new(false);
static mut DATA: u64 = 0;

// SeqCst everywhere -- correct, but pays for a total order nothing here needs
fn producer() {
    unsafe { DATA = 42; }
    READY.store(true, Ordering::SeqCst);
}

fn consumer() -> Option<u64> {
    if READY.load(Ordering::SeqCst) {
        Some(unsafe { DATA })
    } else {
        None
    }
}
```

## Good Example
```rust
use std::sync::atomic::{AtomicBool, AtomicU64, Ordering};

// Relaxed: no ordering relative to other memory -- fine for independent counters
static COUNTER: AtomicU64 = AtomicU64::new(0);
fn increment() {
    COUNTER.fetch_add(1, Ordering::Relaxed);
}

// Acquire/Release: paired handoff -- producer writes payload THEN sets the
// flag with Release; consumer's Acquire load is then guaranteed to see it.
static READY: AtomicBool = AtomicBool::new(false);
static VALUE: AtomicU64 = AtomicU64::new(0);

fn producer(value: u64) {
    VALUE.store(value, Ordering::Relaxed);
    READY.store(true, Ordering::Release);
}

fn consumer() -> Option<u64> {
    if READY.load(Ordering::Acquire) {
        Some(VALUE.load(Ordering::Relaxed))
    } else {
        None
    }
}
```

## Notes
- `Relaxed`: atomic but unordered relative to other memory — counters, stats.
- `Acquire`/`Release`: a load-store pair that hands off one write — the common case for flags and single-producer publication.
- `AcqRel`: read-modify-write ops (`compare_exchange`) acting as both Acquire and Release.
- `SeqCst`: reserve for algorithms needing one global order across *multiple* independent atomics (e.g., Dekker-style mutual exclusion) — most code never needs it.
- Verify ordering choices exhaustively with the `loom` crate, which explores every interleaving the C11 memory model permits.

## References
- [own-mutex-interior](own-mutex-interior.md)
- [The Rustonomicon: Atomics](https://doc.rust-lang.org/nomicon/atomics.html)
