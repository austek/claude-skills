---
title: Use tokio::sync::mpsc for Task-to-Task Message Queues
impact: HIGH
impactDescription: Avoids blocking the executor thread that std::sync::mpsc causes in async code
tags: [async, mpsc, channels, tokio]
---

# Use tokio::sync::mpsc for Task-to-Task Message Queues [HIGH]

## Description
`std::sync::mpsc` blocks the calling OS thread on `recv()` until a message arrives. Call that from inside an async task and the block lands on one of the runtime's worker threads, freezing every other task scheduled on it — not just the caller. `tokio::sync::mpsc` provides the same multi-producer, single-consumer shape but with an async `send`/`recv` that yields the thread back to the scheduler while waiting, plus bounded capacity for backpressure and cheap `Sender` cloning for fan-in from many producer tasks. It is the default channel for task-to-task communication in async Rust; reach for `broadcast`, `watch`, or `oneshot` only when the mpsc shape doesn't fit.

## Bad Example
```rust
use std::sync::mpsc; // wrong channel for async code

let (tx, rx) = mpsc::channel();

tokio::spawn(async move {
    tx.send("hello").unwrap();
});

tokio::spawn(async move {
    let msg = rx.recv().unwrap(); // blocks the executor thread, not just this task
});
```

## Good Example
```rust
use tokio::sync::mpsc;

let (tx, mut rx) = mpsc::channel::<String>(100); // bounded: backpressure built in

tokio::spawn(async move {
    tx.send("hello".to_string()).await.unwrap();
});

tokio::spawn(async move {
    while let Some(msg) = rx.recv().await {
        println!("received: {msg}");
    }
});
```

## Notes
- `Sender::clone()` is cheap and is how multiple producer tasks share one channel; the receive loop ends naturally once every clone (including the original) is dropped and `recv()` returns `None`.
- An actor pattern — a command enum carrying a `oneshot::Sender` for the reply — turns `mpsc` into a request/response channel: the receiving task owns all mutable state and never needs a lock.
- `tx.reserve().await` reserves a guaranteed send slot before the message is built, useful when constructing the message is itself expensive and shouldn't happen only to find the channel full.
- `Sender::downgrade()` produces a `WeakSender` that doesn't keep the channel open by itself — useful for an optional producer that shouldn't prevent the channel from closing when every real producer is gone.
- Bounded vs. unbounded is a separate decision from `std` vs. `tokio`; default to bounded regardless, since an unbounded channel just moves the blocking problem into unbounded memory growth instead.

## References
- [async-bounded-channel](async-bounded-channel.md)
- [async-oneshot-response](async-oneshot-response.md)
- [async-broadcast-pubsub](async-broadcast-pubsub.md)
