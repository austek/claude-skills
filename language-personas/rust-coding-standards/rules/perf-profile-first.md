---
title: Profile Before Optimizing — Don't Guess Where the Time Goes
impact: HIGH
impactDescription: Redirects optimization effort at the code that's actually slow instead of the code that looks slow
tags: [performance, profiling, methodology, flamegraph]
---

# Profile Before Optimizing — Don't Guess Where the Time Goes [HIGH]

## Description
Intuition about where a program spends its time is wrong often enough that it shouldn't be trusted as the basis for optimization work — the clone that "feels expensive" might be 1% of runtime while the innocuous-looking function two calls away is 95% of it. A profiler (`cargo flamegraph`, `perf`, Instruments on macOS) answers the question directly instead of by guesswork: it shows exactly which functions consume wall-clock time in a real, representative run, which turns "I think this is slow" into "this specific call is 60% of the total." Optimizing before that measurement risks spending real engineering time making an already-fast function marginally faster while the actual bottleneck sits untouched.

## Bad Example
```rust
fn process(data: &[Item]) -> Vec<Output> {
    // Assumption: "the clone must be the slow part"
    let cloned: Vec<_> = data.iter().cloned().collect();
    // Reality, unmeasured: 99% of the time is actually spent here
    cloned.iter().map(|x| expensive_computation(x)).collect()
}
```

## Good Example
```rust
// 1. cargo flamegraph --bin myapp -- <args>
// 2. Flamegraph shows expensive_computation() as ~95% of total time,
//    the clone as ~1% — confirmed, not assumed.
// 3. Optimize the confirmed hot spot, leave the clone alone.
fn process(data: &[Item]) -> Vec<Output> {
    let cloned: Vec<_> = data.iter().cloned().collect(); // fine, measured as cheap
    cloned.par_iter().map(|x| expensive_computation(x)).collect() // the real target
}
```

## Notes
- `cargo flamegraph` is the lowest-friction starting point on most platforms; `perf record -g` plus `perf report` (or piped through `inferno`) gives the same information with finer control on Linux; Instruments' Time Profiler covers macOS.
- A wide bar in a flamegraph means time spent, not depth — look for the widest bars first, and pay particular attention to unexpected `malloc`/`free`/`memcpy` frames, which usually point at an allocation or copy nobody intended to be in the hot path.
- The workflow that actually finds real wins: write correct code, add benchmarks for anything suspected to be hot, profile under load resembling production, fix the confirmed bottleneck, measure again to confirm the fix helped, and stop once further profiling shows no single function dominating.

## References
- [opt-lto-release](opt-lto-release.md)
- [test-criterion-bench](../../rust-testing/rules/test-criterion-bench.md)
- [anti-premature-optimize](anti-premature-optimize.md)
