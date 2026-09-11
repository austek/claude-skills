---
title: Tune [profile.release] Instead of Shipping Cargo's Defaults
impact: MEDIUM
impactDescription: A tuned release profile commonly buys 10-40% over cargo's out-of-the-box settings
tags: [performance, cargo, build-configuration, release]
---

# Tune [profile.release] Instead of Shipping Cargo's Defaults [MEDIUM]

## Description
Cargo's default `[profile.release]` is tuned to keep `cargo build --release` reasonably fast for everyday iteration, not to extract maximum runtime performance — `lto = false` and `codegen-units = 16` both trade some amount of cross-crate and cross-unit optimization for parallel, faster compilation. For a binary that's actually shipped to production, that trade generally runs backwards: compile time happens once per release, while runtime performance happens on every execution. A profile that sets `lto = "fat"`, `codegen-units = 1`, `panic = "abort"`, and `strip = true` gives the optimizer the whole program to work with at once, at the cost of a noticeably slower release build.

## Bad Example
```toml
# Cargo.toml — cargo's defaults, tuned for compile speed, not runtime speed
[profile.release]
opt-level = 3
lto = false
codegen-units = 16
```

## Good Example
```toml
# Cargo.toml — tuned for a shipped binary
[profile.release]
opt-level = 3
lto = "fat"        # whole-program link-time optimization
codegen-units = 1  # one unit = no cross-unit optimization barriers
panic = "abort"    # smaller binary, no unwind tables
strip = true       # drop symbols from the shipped artifact
```

## Notes
- `lto` and `codegen-units` each have enough nuance to warrant their own treatment — see `opt-lto-release` for the thin-vs-fat trade-off and `opt-codegen-units` for what setting `1` actually costs in compile time.
- A separate named profile (`[profile.release-dev]` inheriting from `release` with `lto = false` restored) is worth keeping around specifically so day-to-day local `--release` builds don't have to eat the full fat-LTO compile time every iteration.
- `panic = "abort"` removes the ability to catch a panic with `catch_unwind` — fine for most binaries, but not for anything relying on panic recovery (some plugin/FFI hosts, or a server that isolates panicking request handlers) to stay running after a single failure.

## References
- [opt-lto-release](opt-lto-release.md)
- [opt-codegen-units](opt-codegen-units.md)
- [opt-pgo-profile](opt-pgo-profile.md)
