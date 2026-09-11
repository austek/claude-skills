---
title: Never Hold a Mutex/RwLock Guard Across an Await Point
impact: CRITICAL
impactDescription: Prevents deadlocks and task starvation under concurrent load
tags: [async, mutex, rwlock, deadlock]
---

# Never Hold a Mutex/RwLock Guard Across an Await Point [CRITICAL]

## Description
A guard held across an `.await` stays locked for the entire time the task is suspended — which, under load or slow I/O, can be milliseconds to seconds. Every other task that needs the same lock blocks for that whole window, and if the awaited operation itself ever depends (directly or transitively) on that lock being free, the result is a deadlock rather than just contention. The fix is always the same shape: do the async work first, then take the lock only for the brief synchronous update, so the critical section never spans a suspension point.

## Bad Example
```rust
use tokio::sync::Mutex;

async fn bad_update(state: &Mutex<State>) {
    let mut guard = state.lock().await;
    // Lock is held for the full duration of this await — every other
    // task waiting on `state` blocks until this network call returns.
    let data = fetch_from_network().await;
    guard.value = data;
}
```

## Good Example
```rust
use tokio::sync::Mutex;

async fn good_update(state: &Mutex<State>) {
    // Async work happens with no lock held.
    let data = fetch_from_network().await;
    // Lock scope covers only the synchronous write.
    state.lock().await.value = data;
}
```

## Notes
- Prefer `std::sync::Mutex` over `tokio::sync::Mutex` for any critical section that never spans an `.await` — it's simpler and faster, and reaching for the async version at all is usually a sign the critical section is too wide.
- When data must be read before an async operation, extract it in a scoped block so the guard drops before the `.await`: `let id = { state.lock().await.id.clone() };` followed by the async call.
- Message passing (send the update to an owning task over `mpsc` instead of writing through a shared lock) removes the lock from the async path entirely, at the cost of an extra hop.
- `RwLock` doesn't change the rule — even a read guard held across an `.await` blocks every writer waiting behind it, and a common bug is a reader that upgrades to expecting a write later without ever dropping the read guard.
- A held guard that never gets released because the future holding it is dropped mid-poll (cancellation) is a related but distinct failure mode — see the companion rule on cancellation-safety in `select!`.

## References
- [async-spawn-blocking](async-spawn-blocking.md)
- [async-clone-before-await](async-clone-before-await.md)
- [async-cancel-safety](async-cancel-safety.md)
- [own-mutex-interior](own-mutex-interior.md)
