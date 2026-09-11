---
title: Use oneshot for Single Request-Response Replies
impact: MEDIUM
impactDescription: Replaces polling or misused mpsc with a channel built for exactly one value
tags: [async, oneshot, request-response, tokio]
---

# Use oneshot for Single Request-Response Replies [MEDIUM]

## Description
When a task sends one request and needs exactly one reply, `tokio::sync::oneshot` fits precisely: it's a single-use channel with no buffering and no clone overhead, and it's consumed on use so there's no way to accidentally read a second, unrelated value from it. The common alternatives are worse fits — an `mpsc::channel(1)` for one reply keeps the channel alive and open to reuse it can't hold, and polling a shared `Arc<Mutex<Option<T>>>` with a sleep loop burns CPU and adds latency proportional to the poll interval. Paired with `mpsc`, `oneshot` gives a clean request/reply shape: the request carries its own reply `Sender`, and the caller just awaits the matching `Receiver`.

## Bad Example
```rust
// A shared, polled Option in place of a proper reply channel — wastes
// CPU on the sleep loop and adds latency up to the poll interval.
let result = Arc::new(Mutex::new(None));
send_request(result.clone()).await;
while result.lock().await.is_none() {
    tokio::time::sleep(Duration::from_millis(10)).await;
}
```

## Good Example
```rust
use tokio::sync::oneshot;

let (tx, rx) = oneshot::channel::<Response>();
send_request(Request { data, reply: tx }).await;
// Consumed on receipt — no reuse, no polling.
let response = rx.await?;
```

## Notes
- Sending on a `oneshot::Sender` whose `Receiver` was dropped returns `Err` with the value back; receiving on a `Receiver` whose `Sender` was dropped without sending returns `Err(RecvError)` — both are ordinary, expected outcomes to match on, not panics.
- `tx.is_closed()` lets the sender skip an expensive computation once it knows the receiver already gave up waiting, instead of computing a result nobody will read.
- Wrap `rx.await` in `tokio::time::timeout` when the responder might never reply — an unbounded await on a `oneshot::Receiver` blocks forever if the sending task panics or hangs before calling `send`.
- The request-carries-its-own-reply-channel pattern (an enum variant holding both the request payload and a `oneshot::Sender` for the reply) is the standard actor-style RPC shape paired with an `mpsc` command channel.
- `oneshot::Sender` is not `Clone` — if more than one reply is conceptually possible, the shape has outgrown `oneshot` and needs `mpsc` or `broadcast` instead.

## References
- [async-mpsc-queue](async-mpsc-queue.md)
- [async-bounded-channel](async-bounded-channel.md)
- [async-select-racing](async-select-racing.md)
