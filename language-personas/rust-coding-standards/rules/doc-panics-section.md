---
title: Document Every Panic Condition in a # Panics Section
impact: HIGH
impactDescription: Warns callers before an undocumented panic reaches production
tags: [documentation, panics, rustdoc, api-design]
---

# Document Every Panic Condition in a # Panics Section [HIGH]

## Description
A function's signature says nothing about whether it can panic — `fn get(&self, index: usize) -> &T` looks total, but an out-of-bounds index aborts the calling thread. That gap between what the signature promises and what the function actually does is exactly what a `# Panics` section closes: it tells a caller, in writing, the precondition they must uphold to avoid a crash. This matters most in contexts where a panic is expensive or impossible to recover from — across an FFI boundary, inside a `no_std` target, or in a service where one unhandled panic takes down a whole request-handling thread. Silence on this point forces every caller to either read the implementation or find out at runtime.

## Bad Example
```rust
/// Returns the element at the given index.
pub fn get(&self, index: usize) -> &T {
    &self.data[index] // undocumented: panics out of bounds
}
```

## Good Example
```rust
/// Returns the element at the given index.
///
/// # Panics
///
/// Panics if `index >= self.len()`. For a non-panicking alternative,
/// see [`try_get`](Self::try_get).
pub fn get(&self, index: usize) -> &T {
    &self.data[index]
}

/// Returns the element at the given index, or `None` if it's out of bounds.
pub fn try_get(&self, index: usize) -> Option<&T> {
    self.data.get(index)
}
```

## Notes
- Index access, integer division, `.unwrap()`/`.expect()`, and several slice methods (`split_at`, `chunks_exact`) all have panic conditions baked into the standard library — any of them called inside a public function without a guard carries that panic outward and needs documenting.
- When a panic represents a genuine programming error rather than a runtime condition (an invalid argument that should never occur in correct code), say so explicitly — it explains why the API panics instead of returning `Result`.
- Point to a non-panicking alternative (`get` vs. `try_get`, `unwrap` vs. `checked_*`) in the same section whenever one exists, so the caller doesn't have to go looking for it separately.

## References
- [doc-errors-section](doc-errors-section.md)
- [doc-safety-section](doc-safety-section.md)
- [err-result-over-panic](err-result-over-panic.md)
