---
title: Use JoinSet for Dynamic Collections of Spawned Tasks
impact: MEDIUM
impactDescription: Enables results-as-completed processing and guarantees abort-on-drop
tags: [async, joinset, tasks, tokio]
---

# Use JoinSet for Dynamic Collections of Spawned Tasks [MEDIUM]

## Description
Collecting `JoinHandle`s in a `Vec` and awaiting them with `join_all` works, but it's a hand-rolled abstraction over a problem Tokio already solves better. `JoinSet` manages a dynamic set of spawned tasks: it yields each result as that task completes rather than only once all are done, lets tasks be added while others are still running, and — critically — aborts every remaining task automatically when the `JoinSet` itself is dropped. A bare `Vec<JoinHandle<_>>` gives none of that: dropping the vec detaches the handles without cancelling the tasks, and `join_all` only returns after every task finishes, in submission order, even when an early one already failed.

## Bad Example
```rust
// Vec<JoinHandle<_>> — waits for all in submission order, not as they
// complete, and dropping the vec doesn't cancel the still-running tasks.
let mut handles: Vec<JoinHandle<Result<Data>>> = Vec::new();
for url in urls {
    handles.push(tokio::spawn(fetch(url)));
}
let results = futures::future::join_all(handles).await;
```

## Good Example
```rust
use tokio::task::JoinSet;

let mut set = JoinSet::new();
for url in urls {
    set.spawn(fetch(url.clone()));
}

// Each result is available as soon as that task finishes.
while let Some(result) = set.join_next().await {
    match result {
        Ok(Ok(data)) => process(data),
        Ok(Err(e)) => log::error!("task failed: {e}"),
        Err(e) => log::error!("task panicked: {e}"),
    }
}
```

## Notes
- Dropping a `JoinSet` aborts every task still running in it — the single biggest correctness win over a `Vec<JoinHandle>`, where a dropped handle only detaches and the task keeps running unsupervised.
- `join_next()` returns results in completion order, not submission order; if the caller needs to know which input a result belongs to, spawn a future that returns `(index, result)` and use the index to reassemble order.
- `set.abort_all()` cancels every outstanding task on demand — pair it with a subsequent `while set.join_next().await.is_some() {}` to drain the aborted-task results before proceeding.
- `set.len()` combined with `if` guards inside `select!` lets a worker pool cap concurrency: accept new work only while `set.len() < max_concurrent`, and drain completions in the same loop.
- `Ok(Err(e))` (the task ran and returned an application error) and `Err(e)` (the task panicked or was cancelled — check `e.is_panic()`) are distinct outer/inner layers; conflating them loses the distinction between "failed" and "crashed."

## References
- [async-join-parallel](async-join-parallel.md)
- [async-cancellation-token](async-cancellation-token.md)
- [async-try-join](async-try-join.md)
