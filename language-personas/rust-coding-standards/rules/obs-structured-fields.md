---
title: Record Values as Structured Fields, Not Interpolated Into the Message
impact: MEDIUM
impactDescription: Makes every value filterable and chartable in an aggregator without regex parsing
tags: [observability, tracing, structured-logging]
---

# Record Values as Structured Fields, Not Interpolated Into the Message [MEDIUM]

## Description
`info!("processed {n} items for user {id} in {ms}ms", n, id, ms)` reads fine in a terminal, but once that line reaches Loki, Elasticsearch, or an OpenTelemetry backend, `n`, `id`, and `ms` no longer exist as separate data — they're baked into one opaque string that can only be queried by regex, if at all. `tracing`'s field syntax keeps each value discrete: `field = value` records a typed primitive directly, `%expr` records via `Display`, `?expr` records via `Debug` for anything more complex, and the message string is left to hold only a short, stable description that makes sense on its own without the field values substituted in.

## Bad Example
```rust
use tracing::info;

fn process_batch(user_id: u64, items: usize, elapsed_ms: u64) {
    // user_id, items, elapsed_ms are now unqueryable text
    info!("processed {items} items for user {user_id} in {elapsed_ms}ms");
}
```

## Good Example
```rust
use tracing::info;

fn process_batch(user_id: u64, items: usize, elapsed_ms: u64) {
    info!(
        user.id = user_id,
        items,
        elapsed_ms,
        "batch processed" // stable message, values live as separate fields
    );
}
```

## Notes
- Prefer `%expr` over `?expr` when a type has a clean `Display` impl (IDs, paths, URLs) — JSON-formatting backends serialize `Debug` output inconsistently across types, while `Display` output is predictable.
- Bare `field` is shorthand for `field = field` when the field name matches the variable name in scope — useful for cutting boilerplate on the common case.
- Namespaced field names (`user.id`, `http.status`, `db.query`) align with OpenTelemetry semantic conventions and make cross-service correlation easier once traces leave a single process — pick that convention over ad hoc names like `uid` or `userId`.

## References
- [obs-tracing-over-log](obs-tracing-over-log.md)
- [obs-no-sensitive-data](obs-no-sensitive-data.md)
- [obs-error-chain](obs-error-chain.md)
