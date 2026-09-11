---
title: Wrap Benchmark Inputs and Outputs in black_box
impact: HIGH
impactDescription: Without it a benchmark can measure zero real work and report a fabricated speedup
tags: [performance, benchmarking, criterion, optimization]
---

# Wrap Benchmark Inputs and Outputs in black_box [HIGH]

## Description
An optimizing compiler's whole job is to remove work that doesn't affect observable output, and a benchmark loop that computes a value nobody reads is exactly the kind of work it's designed to eliminate — the function call, the loop, sometimes the entire computation can vanish at compile time, leaving a benchmark that measures how fast the CPU does nothing. `std::hint::black_box` (stable in the standard library since Rust 1.66, and re-exported by `criterion`) tells the optimizer to treat a value as opaque: it can't be constant-folded on the way in, and it can't be proven dead on the way out. Skipping it doesn't make a benchmark merely a bit inaccurate — it can make the reported number meaningless, since the "optimized" version might just be running an empty loop faster than the "unoptimized" one runs real work.

## Bad Example
```rust
use criterion::{criterion_group, criterion_main, Criterion};

fn benchmark(c: &mut Criterion) {
    c.bench_function("compute", |b| {
        b.iter(|| {
            expensive_computation(42); // constant input, unused output — may vanish entirely
        });
    });
}
```

## Good Example
```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion};

fn benchmark(c: &mut Criterion) {
    c.bench_function("compute", |b| {
        b.iter(|| {
            // black_box on the input blocks constant folding;
            // black_box on the output blocks dead-code elimination.
            black_box(expensive_computation(black_box(42)))
        });
    });
}
```

## Notes
- Both ends need protecting — hiding only the output still lets the compiler treat `42` as a compile-time constant and precompute as much as it can before the "unknown" boundary; hiding only the input still lets it discard a result nothing reads.
- `black_box(())` inside a loop protects nothing — there has to be an actual value flowing through it; the target is the computed result, not an empty placeholder.
- Setup code (constructing test data, allocating buffers) belongs outside the `b.iter()` closure entirely — only the code actually being measured needs `black_box`, and wrapping setup in it just adds noise to what's being timed.

## References
- [test-criterion-bench](../../rust-testing/rules/test-criterion-bench.md)
- [perf-profile-first](perf-profile-first.md)
- [perf-release-profile](perf-release-profile.md)
