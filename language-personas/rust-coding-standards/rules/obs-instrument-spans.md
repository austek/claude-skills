---
title: Attach Async Work to Spans with #[instrument], Never a Held Guard
impact: HIGH
impactDescription: Keeps concurrent request/task logs correctly correlated instead of silently cross-attributed
tags: [observability, tracing, async, spans]
---

# Attach Async Work to Spans with #[instrument], Never a Held Guard [HIGH]

## Description
A span is what lets an observability backend group every log line and event from one logical operation — one HTTP request, one background job — together, which is the only way to make sense of logs from many concurrent async tasks interleaving on the same output stream. `#[tracing::instrument]` is the safe way to create that grouping for an async function: it wraps the whole `Future` so the span stays correctly attached no matter which executor thread eventually polls it. Entering a span manually with `span.enter()` and holding the resulting guard across an `.await` is the trap — the guard has no idea the task might resume on a different thread, so events after the `.await` can end up attributed to whatever span happens to be active on that thread instead of the one intended.

## Bad Example
```rust
use tracing::{span, Level};

async fn fetch_user(user_id: u64) -> Result<String, String> {
    let span = span!(Level::INFO, "fetch_user", user_id);
    let _guard = span.enter(); // held across the .await below — wrong

    let result = some_async_db_call(user_id).await;
    tracing::info!("fetched user");
    result
}
# async fn some_async_db_call(_id: u64) -> Result<String, String> { Ok("x".into()) }
```

## Good Example
```rust
use tracing::instrument;

// #[instrument] wraps the future correctly; no guard to misuse.
#[instrument(skip(db), fields(user.id = user_id))]
async fn fetch_user(user_id: u64, db: &DbPool) -> Result<String, DbError> {
    tracing::info!("fetching user");
    let user = db.query_user(user_id).await?;
    tracing::info!(username = %user.name, "user fetched");
    Ok(user.name)
}
# struct DbPool;
# struct DbError;
# struct DbUser { name: String }
# impl DbPool { async fn query_user(&self, _id: u64) -> Result<DbUser, DbError> { Ok(DbUser { name: "x".into() }) } }
```

## Notes
- When a span's name needs to be built at runtime (not known at compile time the way `#[instrument]` needs it), build the span manually and attach it with `.instrument(span)` on the future rather than entering it — the future combinator handles the cross-await reattachment correctly.
- `skip(arg)` or `skip_all` on `#[instrument]` keeps large types (connection pools, byte buffers) and sensitive values out of the auto-captured span fields; `fields(key = value)` adds fields beyond what the function's own arguments provide.
- Spans nest automatically, so a child span entered inside a parent's scope preserves that parent-child relationship in trace visualizations (Jaeger, Tempo) without any extra wiring — this only works reliably when the nesting itself follows the same guard-vs-`.instrument()` rule at every level.

## References
- [obs-structured-fields](obs-structured-fields.md)
- [obs-no-sensitive-data](obs-no-sensitive-data.md)
- [async-no-lock-await](async-no-lock-await.md)
