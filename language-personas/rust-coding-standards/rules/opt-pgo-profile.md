---
title: Use Profile-Guided Optimization for Maximum Performance
impact: LOW
impactDescription: Can add 10-30% beyond LTO by optimizing around real runtime behavior
tags: [optimization, pgo, profiling, build-config]
---

# Use Profile-Guided Optimization for Maximum Performance [LOW]

## Description
Profile-Guided Optimization (PGO) feeds the compiler real runtime behavior instead of static heuristics: build an instrumented binary, run it against representative workloads, then rebuild using the collected profile so the compiler optimizes actually-hot paths aggressively and deprioritizes actually-cold ones. The gain — commonly cited at 10-30% on top of standard optimizations — depends entirely on how representative the profiling workload is; profiling with unrealistic microbenchmarks yields hints that mislead the compiler on production traffic. Reserve PGO for production deployments with stable, well-understood workload patterns — it adds real process complexity for a build that most projects don't need.

## Bad Example
```bash
# Using a synthetic microbenchmark to gather the PGO profile
RUSTFLAGS="-Cprofile-generate=/tmp/pgo-data" cargo build --release
./target/release/my_app --tight-loop-microbench  # doesn't reflect real traffic
llvm-profdata merge -o /tmp/pgo-data/merged.profdata /tmp/pgo-data
RUSTFLAGS="-Cprofile-use=/tmp/pgo-data/merged.profdata" cargo build --release
```

## Good Example
```bash
# Instrumented build
RUSTFLAGS="-Cprofile-generate=/tmp/pgo-data" cargo build --release

# Representative workloads: real customer data samples, typical request mixes
./target/release/my_app < test_fixtures/typical_traffic.txt
./target/release/my_app < test_fixtures/stress_case.txt

# Merge and rebuild with the collected profile
llvm-profdata merge -o /tmp/pgo-data/merged.profdata /tmp/pgo-data
RUSTFLAGS="-Cprofile-use=/tmp/pgo-data/merged.profdata" cargo build --release
```

## Notes
- The three steps are always: instrument, profile against representative workloads, rebuild using the merged profile data.
- Profile with real customer data samples or synthetic-but-realistic traffic, never an artificial microbenchmark that stresses one operation in isolation.
- PGO pairs well with `lto = "fat"` and `codegen-units = 1` — apply those first, since PGO amplifies existing cross-crate optimization.
- BOLT can be layered on top of a PGO build for further post-link reordering, typically adding another 5-15%.

## References
- [opt-lto-release](opt-lto-release.md)
- [opt-codegen-units](opt-codegen-units.md)
