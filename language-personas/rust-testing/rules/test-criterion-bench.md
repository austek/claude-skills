---
title: Benchmark with Criterion, Not Manual Timing
impact: MEDIUM
impactDescription: Eliminates noise-driven false conclusions from ad-hoc Instant::now() timing
tags: [testing, benchmarking, criterion, performance]
---

# Benchmark with Criterion, Not Manual Timing [MEDIUM]

## Description
Wrapping a call in `Instant::now()` / `elapsed()` and eyeballing the number tells you almost nothing reliable: one run captures CPU frequency scaling, cache warmth, OS scheduling noise, and whatever else happened to be running at that instant, with no way to tell signal from noise. The `criterion` crate runs the target code for a warm-up period, takes enough timed iterations to build a statistical distribution, flags outliers, and reports a confidence interval instead of a single number — and it can diff two runs to tell you, with a stated confidence level, whether a change actually made things faster or slower. That's the difference between "I think this is quicker" and a number you can put in a PR description.

## Bad Example
```rust
// Manual timing: one sample, no warm-up, no statistics.
let start = std::time::Instant::now();
let result = fibonacci(20);
println!("{:?}", start.elapsed());
// `result` is unused, so an optimizing compiler may delete the call entirely.
```

## Good Example
```rust
// benches/my_benchmark.rs
use criterion::{black_box, criterion_group, criterion_main, Criterion};

fn fibonacci(n: u64) -> u64 {
    match n {
        0 => 0,
        1 => 1,
        n => fibonacci(n - 1) + fibonacci(n - 2),
    }
}

fn bench_fibonacci(c: &mut Criterion) {
    c.bench_function("fib 20", |b| b.iter(|| fibonacci(black_box(20))));
}

criterion_group!(benches, bench_fibonacci);
criterion_main!(benches);
```

```toml
[dev-dependencies]
criterion = "0.5"

[[bench]]
name = "my_benchmark"
harness = false
```

## Notes
- Wrap the benchmarked input in `black_box(...)` (and, when the result itself is otherwise discarded, the output too) — without it, LLVM can prove the computation's result is never observed and delete the whole call, leaving you benchmarking nothing.
- `c.benchmark_group(...)` lets you compare several implementations of the same operation side by side under one report, and `group.throughput(Throughput::Bytes(n))` converts raw timings into a MB/s figure for I/O- or parsing-heavy code.
- `cargo bench -- --save-baseline main` followed later by `cargo bench -- --baseline main` gives you a before/after comparison anchored to a saved run, which is far more trustworthy than comparing two runs from memory.
- Criterion measures wall-clock performance of already-correct code. Verify correctness first — a fast implementation of the wrong behavior is not a win.

## References
- [perf-profile-first](../../rust-coding-standards/rules/perf-profile-first.md)
- [perf-black-box-bench](../../rust-coding-standards/rules/perf-black-box-bench.md)
