---
title: Libraries Emit Through tracing/log, Only Binaries Install a Subscriber
impact: HIGH
impactDescription: Prevents one dependency from silently hijacking or breaking the whole process's logging
tags: [observability, tracing, libraries, api-design]
---

# Libraries Emit Through tracing/log, Only Binaries Install a Subscriber [HIGH]

## Description
Installing a global subscriber (`tracing_subscriber::fmt::init()`) or logger (`env_logger::init()`) is process-wide and can only meaningfully happen once. A library that calls it anyway is making a decision that belongs to whoever owns `main` — the binary is the only place with enough visibility to know what output format, destination, and level filtering the whole process actually needs. When a library installs its own subscriber, it either collides with the application's own initialization (a second `init()` call typically panics or is silently ignored) or, if it runs first, quietly takes control of logging away from the application entirely. The correct split, which `log` has enforced from the start and `tracing` carries forward: libraries only emit events and spans; the binary decides what happens to them.

## Bad Example
```rust
// mylib/src/lib.rs
pub fn connect(url: &str) {
    tracing_subscriber::fmt::init(); // library has no business doing this
    tracing::info!(url, "connecting");
}
```

## Good Example
```rust
// mylib/src/lib.rs
pub fn connect(url: &str) {
    tracing::info!(url, "connecting"); // just emit
}

// src/main.rs
fn main() {
    tracing_subscriber::fmt() // the application owns this decision
        .with_env_filter(tracing_subscriber::EnvFilter::from_default_env())
        .init();

    mylib::connect("postgres://localhost/app");
}
```

## Notes
- A library's `Cargo.toml` should depend on `tracing` (or `log`) directly, but keep `tracing-subscriber` (or `env_logger`) out of its non-dev dependencies entirely — pulling it in as a regular dependency invites exactly this mistake even if no code path calls `init()` yet.
- Library tests that need visible output can add `tracing-subscriber` under `[dev-dependencies]` and call `tracing_subscriber::fmt::try_init()` inside `#[test]` functions — the `try_` variant returns an error instead of panicking on a second call, which matters when multiple tests in the same binary each try to initialize it.
- Bridging the two ecosystems is one line, not a library-side workaround: an application depending on crates that emit through `log` can call `tracing_log::LogTracer::init()` once to route those events through the same `tracing` subscriber.

## References
- [obs-tracing-over-log](obs-tracing-over-log.md)
- [obs-levels-filter](obs-levels-filter.md)
