---
title: Enable Link-Time Optimization in Release Builds
impact: MEDIUM
impactDescription: Typically 5-20% faster binaries via cross-crate optimization
tags: [optimization, lto, build-config, release-profile]
---

# Enable Link-Time Optimization in Release Builds [MEDIUM]

## Description
Normal compilation optimizes each crate independently, so opportunities that span crate boundaries — cross-crate inlining, dead code elimination, devirtualization — go unexploited. Link-Time Optimization runs a further optimization pass across the linked whole, at the cost of a substantially slower build. `lto = "thin"` captures most of the benefit for a moderate compile-time cost; `lto = "fat"` (or `true`) pushes further at the slowest build times. As with `codegen-units = 1`, this belongs in the production release profile, not in day-to-day development.

## Bad Example
```toml
# Cargo.toml
[profile.release]
opt-level = 3
# no LTO — cross-crate optimization opportunities are missed
```

## Good Example
```toml
# Cargo.toml
[profile.release]
opt-level = 3
lto = "fat"          # maximum cross-crate optimization
codegen-units = 1    # required for LTO's full effect
panic = "abort"      # smaller binary, no unwind tables
```

## Notes
- `lto = "thin"` compiles faster than `"fat"` and still captures most of the gain — a reasonable default for CI release builds.
- LTO and `codegen-units = 1` are complementary: LTO without a single codegen unit still leaves some optimization on the table.
- Library crates published to crates.io generally should not force LTO in their own profile — it's a decision for the final binary's own `Cargo.toml`, not something a dependency should impose.
- Measure with `hyperfine` on the built binary before and after; the 5-20% figure is typical, not guaranteed for every workload.

## References
- [opt-codegen-units](opt-codegen-units.md)
- [opt-pgo-profile](opt-pgo-profile.md)
- [The Cargo Book: Profiles](https://doc.rust-lang.org/cargo/reference/profiles.html)
