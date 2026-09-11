---
title: Use try_join! for Concurrent Fallible Operations
impact: HIGH
impactDescription: Fails fast on the first error while still running futures concurrently
tags: [async, try-join, error-handling, tokio]
---

# Use try_join! for Concurrent Fallible Operations [HIGH]

## Description
Running several fallible operations with `.await?` in sequence pays their full combined latency even when an early one fails and the rest of the work turns out to be wasted. `tokio::try_join!` runs the futures concurrently like `join!`, but returns `Err` as soon as any one of them fails, without waiting for the still-running ones to finish — fail-fast behavior on top of concurrent execution. Plain `join!` is the wrong tool for fallible futures specifically because it has no concept of failure: it waits for every future regardless of outcome and hands back a tuple of `Result`s that the caller must still unwrap and match individually.

## Bad Example
```rust
// Sequential .await? pays full latency even when an early call fails
// and the rest of the work is never going to be used.
async fn fetch_all() -> Result<(A, B, C)> {
    let a = fetch_a().await?;
    let b = fetch_b().await?;
    let c = fetch_c().await?;
    Ok((a, b, c))
}
```

## Good Example
```rust
use tokio::try_join;

async fn fetch_all() -> Result<(A, B, C)> {
    // Concurrent and fail-fast: returns as soon as any one call errors.
    let (a, b, c) = try_join!(fetch_a(), fetch_b(), fetch_c())?;
    Ok((a, b, c))
}
```

## Notes
- All futures passed to `try_join!` must share one error type (or be mapped to one with `.map_err(Error::from)` beforehand) — this is the price of the fail-fast `?` at the end, unlike `join!` which tolerates mixed `Result` types since it never short-circuits.
- On error, the still-running futures are dropped, not stopped instantly — they stop at their next internal `.await` point, so any cleanup code written after an `.await` inside one of them may never run; use a `Drop` guard for cleanup that must happen unconditionally.
- For a dynamic-length collection, `futures::future::try_join_all` is the equivalent of `try_join!` for a `Vec` of futures rather than a fixed, named set.
- When partial success is acceptable — collect whichever calls succeeded and just log the failures — plain `join_all` followed by `filter_map` over the results fits better than `try_join!`'s all-or-nothing semantics.
- `FuturesUnordered` from the `futures` crate is the right choice when results should be processed as they arrive rather than only once the whole batch resolves, while still supporting fail-fast by returning early from the `while let Some(...)` loop on the first error.

## References
- [async-join-parallel](async-join-parallel.md)
- [async-select-racing](async-select-racing.md)
- [err-question-mark](err-question-mark.md)
