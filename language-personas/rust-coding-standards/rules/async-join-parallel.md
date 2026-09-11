---
title: Use join! to Run Independent Futures Concurrently
impact: HIGH
impactDescription: Cuts latency from the sum of durations to the slowest one
tags: [async, join, concurrency, tokio]
---

# Use join! to Run Independent Futures Concurrently [HIGH]

## Description
Awaiting independent futures one after another pays their full sum in latency: three 100ms calls awaited in sequence take 300ms, even though nothing about them depends on the others' results. `tokio::join!` polls multiple futures on the same task concurrently and returns once all of them complete, so the same three calls take roughly 100ms — the duration of the slowest one. The requirement is that the futures be genuinely independent; `join!` still runs everything to completion regardless of individual failures, so error handling needs `try_join!` instead, and a truly dynamic-length collection needs `futures::future::join_all` rather than the fixed-arity macro.

## Bad Example
```rust
async fn fetch_data() -> (User, Posts, Comments) {
    // Sequential: pays the full sum, ~300ms, though nothing here depends
    // on the result of a prior call.
    let user = fetch_user().await;
    let posts = fetch_posts().await;
    let comments = fetch_comments().await;
    (user, posts, comments)
}
```

## Good Example
```rust
use tokio::join;

async fn fetch_data() -> (User, Posts, Comments) {
    // Concurrent: bounded by the slowest call, ~100ms.
    let (user, posts, comments) = join!(
        fetch_user(),
        fetch_posts(),
        fetch_comments(),
    );
    (user, posts, comments)
}

// Dynamic-length collection: join! requires a fixed set of futures known
// at compile time, so a runtime-sized Vec needs join_all instead.
use futures::future::join_all;

async fn fetch_all_users(ids: &[u64]) -> Vec<User> {
    let futures: Vec<_> = ids.iter().map(|id| fetch_user(*id)).collect();
    join_all(futures).await
}
```

## Notes
- `join!` runs every future to completion and returns a tuple of all results, error or not — it never short-circuits. Use `try_join!` when any failure should stop the others early instead.
- Dependent operations (the second call needs the first one's output) must stay sequential; `join!`ing them either doesn't compile or silently races on state that was assumed to already exist.
- Unbounded concurrency from `join_all` over a large collection can overwhelm a downstream service or exhaust file descriptors — cap it with `futures::stream::StreamExt::buffer_unordered(n)` or a `Semaphore` when the collection size isn't small and fixed.
- Racing to the first result, rather than waiting for all of them, is a different problem — that's what `select!` is for, not `join!`.
- Shared mutable state (an `Arc<Mutex<_>>` written by every joined future) works but reintroduces the lock contention `join!`'s concurrency was meant to avoid; prefer each future returning its own result and merging afterward.

## References
- [async-try-join](async-try-join.md)
- [async-select-racing](async-select-racing.md)
- [async-joinset-structured](async-joinset-structured.md)
