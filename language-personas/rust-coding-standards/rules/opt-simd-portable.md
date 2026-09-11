---
title: Reach for Portable SIMD Before Architecture-Specific Intrinsics
impact: HIGH
impactDescription: 4-8x throughput on suitable numeric workloads
tags: [optimization, simd, vectorization, portability]
---

# Reach for Portable SIMD Before Architecture-Specific Intrinsics [HIGH]

## Description
SIMD processes multiple values per instruction, and suitable algorithms — element-wise numeric transforms, reductions, dot products — see 4x, 8x, or larger speedups from it. On stable Rust, LLVM autovectorizes simple iterator-based loops without any code change; when that isn't enough, the `wide` crate gives explicit, portable SIMD types on stable, while nightly's `std::simd` offers the most control at the cost of instability. Reach for raw architecture intrinsics (`std::arch::x86_64`) only when neither autovectorization nor a portable crate reaches the needed performance, since intrinsics tie the binary to one instruction set and require `unsafe`.

## Bad Example
```rust
// Hand-written scalar loop with early-exit style that can block autovectorization
fn sum(data: &[f32]) -> f32 {
    let mut total = 0.0;
    let mut i = 0;
    while i < data.len() {
        total += data[i];
        i += 1;
    }
    total
}
```

## Good Example
```rust
// Iterator form: LLVM often autovectorizes this on stable Rust
fn sum(data: &[f32]) -> f32 {
    data.iter().sum()
}

// wide crate: explicit, portable SIMD on stable when autovectorization
// isn't reliable enough
use wide::f32x8;

fn scale_and_shift(data: &mut [f32]) {
    for chunk in data.chunks_exact_mut(8) {
        let v = f32x8::from(&*chunk);
        let result = v * f32x8::splat(2.0) + f32x8::splat(1.0);
        chunk.copy_from_slice(&result.to_array());
    }
}
```

## Notes
- Help autovectorization along: prefer iterators over manual indexing, avoid early exits inside the hot loop, and use `chunks_exact` for aligned, fixed-size access.
- The `wide` crate is the practical stable choice for portable SIMD; nightly `std::simd` (`portable_simd` feature) offers more control but no stability guarantee.
- Architecture-specific intrinsics (`std::arch::x86_64`, gated behind `#[target_feature]` and `unsafe`) are the last resort — they require runtime feature detection (`is_x86_feature_detected!`) to stay portable.
- Verify a change actually vectorized with generated assembly (`cargo asm`) rather than assuming — the compiler's decision is a heuristic, not a promise.

## References
- [opt-target-cpu](opt-target-cpu.md)
- [opt-bounds-check](opt-bounds-check.md)
