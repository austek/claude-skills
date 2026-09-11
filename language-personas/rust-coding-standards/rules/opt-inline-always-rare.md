---
title: Reserve #[inline(always)] for Proven Hot Paths
impact: MEDIUM
impactDescription: Overuse bloats binaries and hurts instruction-cache hit rate
tags: [optimization, inlining, inline-always, profiling]
---

# Reserve #[inline(always)] for Proven Hot Paths [MEDIUM]

## Description
`#[inline(always)]` forces the compiler to inline a function regardless of its own heuristics, which normally do a better job weighing code-size growth against call overhead. Annotating everything "to be safe" bloats the binary and can actively hurt performance by pushing hot code out of the instruction cache. The attribute earns its place only on tiny functions in measured hot loops — a hasher's `write`, a SIMD kernel — where a benchmark, not intuition, shows the gain.

## Bad Example
```rust
// Annotating routine accessors and helpers "just in case"
#[inline(always)]
pub fn get_name(&self) -> &str {
    &self.name
}

#[inline(always)]
fn helper(x: i32) -> i32 {
    x + 1
}
```

## Good Example
```rust
// Let the compiler decide for ordinary functions
pub fn get_name(&self) -> &str {
    &self.name
}

// Force inlining only where profiling proved a measurable win
impl Hasher for MyHasher {
    #[inline(always)]
    fn write(&mut self, bytes: &[u8]) {
        self.state = self.state.wrapping_add(bytes.len() as u64);
    }
}
```

## Notes
- Good candidates: single-expression functions called in a tight inner loop, generic functions that benefit from monomorphization, SIMD helper functions.
- `#[inline]` (without `always`) is a weaker hint the compiler may ignore — prefer it as the default when cross-crate inlining is desired but not forced.
- Generic public functions often need at least `#[inline]` to be inlinable across crate boundaries at all, since the body must be available to the calling crate.
- Verify the win with `cargo bloat --release --crates` (binary size) and a `criterion` benchmark, not by reasoning alone.

## References
- [opt-inline-small](opt-inline-small.md)
- [opt-inline-never-cold](opt-inline-never-cold.md)
