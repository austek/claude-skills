---
title: Set codegen-units = 1 for Maximum Release Optimization
impact: LOW
impactDescription: Typically 5-20% faster binaries at the cost of much slower release builds
tags: [optimization, build-config, codegen-units, release-profile]
---

# Set codegen-units = 1 for Maximum Release Optimization [LOW]

## Description
Cargo splits a crate into multiple codegen units by default so LLVM can compile them in parallel, which speeds up builds but blocks optimizations that need to see the whole crate at once — cross-module inlining, dead code elimination, constant propagation. Setting `codegen-units = 1` forces a single unit, letting LLVM optimize globally at the cost of a much slower, non-parallel release build. Reserve it for the final production profile, not for every-day development or CI builds where compile time matters more.

## Bad Example
```toml
# Cargo.toml
[profile.release]
opt-level = 3
# codegen-units defaults to 16 — fast to compile, misses cross-unit optimization
```

## Good Example
```toml
# Cargo.toml
[profile.release]
opt-level = 3
codegen-units = 1  # single unit: LLVM sees and optimizes the whole crate
lto = true         # pairs naturally with codegen-units = 1
```

## Notes
- The trade-off is monotonic: 16 (default) compiles fastest, 1 compiles slowest and runs fastest; pick a value per profile rather than crate-wide.
- Keep `codegen-units` at its default (or raised, e.g. 256) for `[profile.dev]` — this setting only pays off when the binary ships.
- Combine with `lto = "fat"` for the largest effect; codegen-units alone captures only part of the available cross-crate optimization.
- Measure with `hyperfine` on the built binary — the 5-20% figure is a typical range, not a guarantee for every workload.

## References
- [opt-lto-release](opt-lto-release.md)
- [opt-pgo-profile](opt-pgo-profile.md)
- [The Cargo Book: Profiles](https://doc.rust-lang.org/cargo/reference/profiles.html)
