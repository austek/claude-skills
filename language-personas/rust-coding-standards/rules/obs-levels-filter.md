---
title: Pick Log Levels by Urgency and Filter at Runtime with EnvFilter
impact: MEDIUM
impactDescription: Keeps production log volume operator-tunable without a rebuild
tags: [observability, tracing, logging, operations]
---

# Pick Log Levels by Urgency and Filter at Runtime with EnvFilter [MEDIUM]

## Description
A log level is a promise about urgency, and an operator relies on that promise to decide what to look at during an incident versus what to ignore during normal operation. Emitting everything through `info!` — or leaving per-iteration `debug!`/`trace!` detail running at production volume — breaks that promise and drowns real signal in noise expensive aggregators then have to index and store. `tracing_subscriber::EnvFilter` reads `RUST_LOG` at process start and supports comma-separated `target=level` directives, so an operator can raise verbosity for one crate under investigation (`myapp=debug`) while keeping everything else quiet, with no redeploy required.

## Bad Example
```rust
use tracing::info;

fn handle_request(path: &str, body: &[u8]) {
    info!(path, raw = ?body, "handling request"); // full body at info — noisy
    info!("entered handler");                      // lifecycle noise at info
    info!("done handling request");
}
```

## Good Example
```rust
use tracing::{debug, error, info, trace, warn};

fn handle_request(path: &str, body: &[u8]) {
    trace!("entered handler");
    debug!(body_len = body.len(), "parsing body");
    info!(path, "request received");

    match parse_body(body) {
        Ok(n) => info!(items = n, "request processed"),
        Err(e) if is_client_error(&e) => warn!(error = ?e, "malformed request"),
        Err(e) => error!(error = ?e, "unexpected parse failure"),
    }
}
# fn parse_body(_b: &[u8]) -> Result<usize, String> { Ok(0) }
# fn is_client_error(_e: &str) -> bool { false }

fn main() {
    tracing_subscriber::fmt()
        .with_env_filter(
            tracing_subscriber::EnvFilter::try_from_default_env()
                .unwrap_or_else(|_| "info,myapp=debug,hyper=warn".into()),
        )
        .init();
}
```

## Notes
- Rough guide: `error!` for failures needing immediate attention, `warn!` for recoverable anomalies, `info!` for lifecycle milestones, `debug!` for development-time diagnostic detail, `trace!` for per-iteration verbosity nobody wants on by default.
- `try_from_default_env()` with a fallback string, rather than `from_default_env()`, keeps the binary starting normally when `RUST_LOG` is unset or contains a typo, instead of failing at startup.
- Cargo features `max_level_debug` / `release_max_level_info` on the `tracing` dependency compile out call sites below the chosen level entirely — this removes even the branch-check overhead in release builds, not just the output.

## References
- [obs-tracing-over-log](obs-tracing-over-log.md)
- [obs-library-facade](obs-library-facade.md)
