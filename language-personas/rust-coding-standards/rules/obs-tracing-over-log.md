---
title: Reach for tracing Instead of println! or Bare log for Diagnostics
impact: MEDIUM
impactDescription: Unlocks levels, structured fields, and async-aware spans a println! call can never have
tags: [observability, tracing, logging]
---

# Reach for tracing Instead of println! or Bare log for Diagnostics [MEDIUM]

## Description
`println!`/`eprintln!` write unconditionally to a stream with no level, no structure, and no way to silence or filter them short of removing the call — fine for a quick debugging session, a liability in anything that runs in production. The `log` crate fixes the level and filtering gap but still only emits flat message strings with no notion of grouping related events together. `tracing` adds two things `log` doesn't have: structured key-value fields attached directly to each event, and spans — scopes that represent one logical operation and stay correctly attached to it even as an async task moves across `.await` points and worker threads. A `log` feature flag also bridges the two, so existing `log`-emitting dependencies still show up through a `tracing` subscriber without any code changes in those dependencies.

## Bad Example
```rust
fn handle_login(id: u64) {
    println!("user {id} logged in"); // no level, no structure, always fires
}
```

## Good Example
```rust
use tracing::info;

fn handle_login(id: u64) {
    info!(user.id = %id, "user logged in"); // filterable, structured, levelled
}

fn main() {
    tracing_subscriber::fmt::init(); // subscriber setup belongs in the binary
    handle_login(42);
}
```

## Notes
- `%expr` records a field via `Display`, `?expr` via `Debug`, and a bare `field = value` records a typed primitive directly — see `obs-structured-fields` for the full picture.
- Add `tracing = "0.1"` to every crate that wants to emit diagnostics; reserve `tracing-subscriber` for the binary that actually configures output — see `obs-library-facade` for why that split matters.
- `tracing_log::LogTracer::init()` (or the `log` feature on `tracing-subscriber`) routes events from dependencies still using the plain `log` facade into the same `tracing` subscriber, so migrating doesn't require waiting on every dependency to switch first.

## References
- [obs-structured-fields](obs-structured-fields.md)
- [obs-instrument-spans](obs-instrument-spans.md)
- [obs-library-facade](obs-library-facade.md)
