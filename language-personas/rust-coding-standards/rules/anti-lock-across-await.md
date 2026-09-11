---
title: Treat a Lock Guard Held Across .await as an Immediate Red Flag
impact: CRITICAL
impactDescription: With std::sync::Mutex this can deadlock the entire async runtime
tags: [anti-pattern, async, concurrency, deadlock]
---

# Treat a Lock Guard Held Across .await as an Immediate Red Flag [CRITICAL]

## Description
A `MutexGuard` bound to a `let` that stays in scope across an `.await` keeps the lock held for the entire time the task is suspended — which, for an async task, can be an arbitrarily long and unpredictable amount of time, not the few nanoseconds a synchronous critical section normally takes. Any other task that needs the same lock is stuck waiting for the whole suspension to finish. With `std::sync::Mutex` specifically, this is worse than merely slow: that lock isn't async-aware at all, and holding it across an await point on an executor that can run other work on the same thread risks a full deadlock, not just contention. The rule of thumb: a lock guard should never share scope with an `.await` — extract what's needed, drop the guard, then await.

## Bad Example
```rust
use std::sync::Mutex;

async fn update(data: &Mutex<Vec<i32>>) {
    let mut guard = data.lock().unwrap();
    do_async_work().await; // guard is still held here — other lockers wait or deadlock
    guard.push(42);
}
```

## Good Example
```rust
use std::sync::Mutex;

async fn update(data: &Mutex<Vec<i32>>) {
    let snapshot = {
        let guard = data.lock().unwrap();
        guard.last().copied()
    }; // guard dropped before the await

    let result = do_async_work(snapshot).await;

    data.lock().unwrap().push(result); // lock reacquired only after the await
}
```

## Notes
- A `tokio::sync::Mutex` guard held across `.await` doesn't deadlock the runtime the way `std::sync::Mutex` can, but it still blocks every other task waiting on that lock for the full duration of the awaited work — it trades a catastrophic failure mode for a milder but still real one.
- Cloning what's needed out of the guarded data before the await point (see `async-clone-before-await`) sidesteps the whole problem when the data is cheap to clone; restructuring around channels sidesteps it entirely when it isn't.
- `clippy::await_holding_lock` (and `await_holding_refcell_ref` for the analogous `RefCell` case) catches this mechanically — worth setting to `deny` rather than `warn`, since there's essentially never a legitimate reason to hold a lock across an await.

## References
- [async-no-lock-await](async-no-lock-await.md)
- [async-clone-before-await](async-clone-before-await.md)
- [own-mutex-interior](own-mutex-interior.md)
