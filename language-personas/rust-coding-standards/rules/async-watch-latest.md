---
title: Use watch for Broadcasting the Latest Value
impact: MEDIUM
impactDescription: Slow observers skip stale intermediate values instead of lagging or blocking
tags: [async, watch, channels, tokio]
---

# Use watch for Broadcasting the Latest Value [MEDIUM]

## Description
`tokio::sync::watch` is built for the specific case where observers only ever care about the current value, not the sequence of updates that produced it — configuration, connection state, health status. A slow `watch` receiver doesn't lag behind or apply backpressure to the sender: calling `changed().await` after missing several updates simply resolves once, against whatever the latest value now is, silently skipping the intermediate ones. That's the opposite of `broadcast`, which delivers every message and makes a slow receiver either lag (and miss messages once its buffer overflows) or block the sender — the right choice when history matters, but wasted and even harmful work when only the current state does.

## Bad Example
```rust
// broadcast delivers every intermediate config, even when only the
// latest one will ever actually be applied — wasted work, and a slow
// receiver risks lagging and missing updates outright.
let (tx, _) = broadcast::channel::<Config>(100);
```

## Good Example
```rust
use tokio::sync::watch;

let (tx, rx) = watch::channel(Config::default());
let mut rx = rx.clone();

tokio::spawn(async move {
    while rx.changed().await.is_ok() {
        let config = rx.borrow();
        apply_config(&config);
    }
});

tx.send(Config::new())?;
```

## Notes
- Multiple rapid `send`s collapse into one observed change: a receiver that calls `changed().await` after three sends in a row wakes once and sees only the last value — by design, not a bug to work around.
- `borrow()` returns a `Ref` that must not be held across an `.await`; clone the value out first (`rx.borrow().clone()`) if it's needed after a subsequent await point.
- `borrow_and_update()` reads the current value and marks it as seen in one step, which is what most consumer loops actually want over a bare `borrow()` that leaves the "changed" flag untouched.
- `send_if_modified` lets the sender skip notifying receivers entirely when the new value equals the old one, avoiding a wakeup storm for updates that turn out to be no-ops.
- Choose by shape, not habit: `watch` for "only the latest matters," `broadcast` for "every event must be seen by every subscriber," `mpsc` for "each message goes to exactly one consumer."

## References
- [async-broadcast-pubsub](async-broadcast-pubsub.md)
- [async-mpsc-queue](async-mpsc-queue.md)
- [async-cancellation-token](async-cancellation-token.md)
