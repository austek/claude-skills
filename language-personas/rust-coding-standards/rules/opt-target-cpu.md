---
title: Set target-cpu for Known Deployment Targets
impact: MEDIUM
impactDescription: Unlocks CPU features (AVX2, AVX-512) unused by the generic baseline build
tags: [optimization, target-cpu, build-config, simd]
---

# Set target-cpu for Known Deployment Targets [MEDIUM]

## Description
Rust compiles for a generic x86-64 baseline by default — roughly a Sandy-Bridge-era instruction set — so the binary runs everywhere but leaves modern SIMD extensions (AVX2, AVX-512), FMA, and other micro-architectural features on the table. When the deployment target is known, setting `target-cpu` (to `native`, or to a specific microarchitecture for cross-compilation) lets the compiler use those features directly, often improving autovectorization and numeric throughput significantly. This trades portability for performance: a binary built for `native` or a specific CPU can crash with an illegal-instruction fault on an older or different machine, so it's only safe when the build target matches the deployment target.

## Bad Example
```toml
# .cargo/config.toml — nothing set, binary stays on the generic baseline
[build]
# rustflags unset: compiles for x86-64 baseline (SSE2 only)
```

## Good Example
```toml
# .cargo/config.toml — known deployment target
[build]
rustflags = ["-C", "target-cpu=x86-64-v3"]  # AVX2, BMI2 — safe for post-2013 x86-64 servers
```

## Notes
- `target-cpu=native` is appropriate for local builds or a fleet built on identical hardware; never ship it in a binary that runs on unknown machines.
- `x86-64-v2`/`v3`/`v4` are portable feature-level targets (SSE4.2; AVX2+BMI2; AVX-512) — safer than `native` for a known-but-heterogeneous fleet.
- For portable binaries that still want the speedup where available, detect the feature at runtime with `is_x86_feature_detected!` and dispatch to a `#[target_feature(enable = "avx2")]` function.
- Verify what a setting actually enables with `rustc --print cfg -C target-cpu=<value> | grep target_feature` before relying on it.

## References
- [opt-simd-portable](opt-simd-portable.md)
- [opt-lto-release](opt-lto-release.md)
- [opt-codegen-units](opt-codegen-units.md)
