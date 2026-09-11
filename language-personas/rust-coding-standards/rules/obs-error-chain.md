---
title: Log the Full Error Chain Once, at the Layer That Handles It
impact: HIGH
impactDescription: Preserves root-cause context and stops the same failure flooding aggregators N times
tags: [observability, tracing, error-handling, logging]
---

# Log the Full Error Chain Once, at the Layer That Handles It [HIGH]

## Description
Two separate mistakes compound to make error logging worse than useless: logging only an error's top-level `Display` message throws away the `source()` chain that would explain the actual root cause, and logging at every layer an error passes through on its way up the call stack turns one failure into a wall of duplicate log lines with slightly different context at each level. The fix addresses both: propagate with `?` and `.context()`/`.with_context()` through intermediate functions without logging anything there, and log exactly once — with the full chain — at the boundary that actually decides what to do about the failure (an HTTP handler returning a response, a job runner marking a task failed). `anyhow::Error`'s alternate `{:#}` format walks the whole cause chain with `: ` separators; `?err` captures the same information via `Debug` for any error implementing `std::error::Error`.

## Bad Example
```rust
async fn read_from_db(id: u64) -> Result<Vec<u8>, DbError> {
    let result = inner_query(id).await;
    if let Err(ref e) = result {
        tracing::error!("{}", e); // logs only the top message, and logs early
    }
    result
}

async fn fetch_data(id: u64) -> Result<Vec<u8>, DbError> {
    let result = read_from_db(id).await;
    if let Err(ref e) = result {
        tracing::error!("{}", e); // same failure, logged a second time
    }
    result
}
```

## Good Example
```rust
use anyhow::{Context, Result};

// Intermediate layers only add context and propagate — no logging.
async fn read_from_db(id: u64) -> Result<Vec<u8>> {
    inner_query(id)
        .await
        .with_context(|| format!("reading record {id}"))
}

async fn fetch_data(id: u64) -> Result<Vec<u8>> {
    read_from_db(id).await.context("fetch_data failed")
}

// The handling boundary logs once, with the full chain.
async fn handle_request(id: u64) -> Result<(), String> {
    match fetch_data(id).await {
        Ok(data) => { process(data); Ok(()) }
        Err(err) => {
            tracing::error!(error = %format!("{err:#}"), "request failed");
            Err("internal error".into())
        }
    }
}
# async fn inner_query(_id: u64) -> anyhow::Result<Vec<u8>> { Ok(vec![]) }
# fn process(_data: Vec<u8>) {}
```

## Notes
- `{err:#}` on an `anyhow::Error` prints every layer of context added via `.context()`, separated by `: ` — that's the human-readable equivalent of walking `source()` manually.
- A background task that swallows an error rather than returning it (no caller left to handle it) is the one legitimate spot to log outside the handling boundary — use `warn!`, not `error!`, to mark that the failure was intentionally absorbed rather than surfaced.
- `tracing-error`'s `SpanTrace` captures the active span context at the point an error originates and can be attached to a custom error type, giving the eventual handler-level log line span context from deep inside the call stack without threading it through manually.

## References
- [err-context-chain](err-context-chain.md)
- [err-source-chain](err-source-chain.md)
- [anti-empty-catch](anti-empty-catch.md)
