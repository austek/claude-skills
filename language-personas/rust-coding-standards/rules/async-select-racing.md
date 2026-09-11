---
title: Use select! to Race Futures and Cancel the Losers
impact: HIGH
impactDescription: Replaces error-prone manual timeout/cancellation loops with structured racing
tags: [async, select, racing, tokio]
---

# Use select! to Race Futures and Cancel the Losers [HIGH]

## Description
Some operations need "whichever finishes first," not "all of them" — an operation racing a timeout, a primary source racing a fallback, or work racing a shutdown signal. `tokio::select!` polls several futures concurrently, runs the handler for whichever branch completes first, and drops every other branch at that point. That drop is real cancellation, not a soft signal: the losing futures stop being polled from that instant, and any internal state they were accumulating is discarded unless the future in question is designed to survive being dropped mid-poll. Hand-rolled alternatives — a manual elapsed-time check inside a loop, or racing with a boolean flag — get the timeout arithmetic or the cancellation propagation wrong in ways `select!` handles correctly by construction.

## Bad Example
```rust
// Sequential fallback: not a race — always pays fetch_primary's full
// latency before even starting the fallback.
async fn fetch_with_fallback() -> Data {
    match fetch_primary().await {
        Ok(data) => data,
        Err(_) => fetch_fallback().await.unwrap(),
    }
}
```

## Good Example
```rust
use tokio::select;

async fn fetch_with_timeout() -> Result<Data, Error> {
    select! {
        result = fetch_data() => result,
        _ = tokio::time::sleep(Duration::from_secs(5)) => Err(Error::Timeout),
    }
}
```

## Notes
- By default `select!` picks pseudo-randomly among branches that are ready simultaneously; add `biased;` as the first line inside the macro to check branches in listed order instead, needed whenever one branch (like a shutdown signal) must take priority.
- A losing branch is dropped at the point `select!` decides the race, not at its next natural yield point — any future accumulating state internally (like `read_exact`) loses that state unless it's proven cancel-safe.
- `if <condition>` guards on a branch (`msg = rx.recv(), if enabled => ...`) disable that branch without removing it from the macro, useful for toggling a branch on and off across loop iterations without restructuring the `select!`.
- An `else` branch runs when every other branch is disabled by its guard — omitting it causes a panic if that state is ever reached, so include it whenever guards can plausibly disable everything at once.
- For a dynamic-length set of futures to race (not a fixed set known at compile time), `futures::future::select_all` is the equivalent — `select!`'s branches must be enumerable in the macro invocation itself.

## References
- [async-cancel-safety](async-cancel-safety.md)
- [async-cancellation-token](async-cancellation-token.md)
- [async-join-parallel](async-join-parallel.md)
- [async-bounded-channel](async-bounded-channel.md)
