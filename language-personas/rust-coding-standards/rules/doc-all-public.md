---
title: Write a Doc Comment for Every Public Item
impact: MEDIUM
impactDescription: Unblocks cargo doc as a real reference instead of empty stubs
tags: [documentation, api-design, public-api, rustdoc]
---

# Write a Doc Comment for Every Public Item [MEDIUM]

## Description
Everything exported across a crate boundary is a promise to whoever depends on it, and a `///` comment is how that promise gets written down somewhere other than the implementation. Skip the comment and a caller's only recourse is reading the function body — fine for a small internal tool, a real cost for a library where the source may not even be open. `cargo doc` turns these comments into the HTML page most users will actually read before they read any code, so an undocumented public item is effectively a blank page in that reference. `#![warn(missing_docs)]` (or the equivalent `[workspace.lints.rust]` entry) turns the omission into a build-time signal instead of something that's only noticed later.

## Bad Example
```rust
pub struct RetryPolicy {
    pub max_attempts: u32,
    pub backoff: Duration,
}

pub fn execute_with_retry(policy: RetryPolicy) -> Result<Output, Error> {
    // ...
}
```

## Good Example
```rust
/// Controls how a failed operation is retried before giving up.
pub struct RetryPolicy {
    /// Total number of attempts, including the first.
    pub max_attempts: u32,
    /// Delay applied between attempts.
    pub backoff: Duration,
}

/// Runs `operation` under the given [`RetryPolicy`], retrying on failure.
///
/// # Errors
///
/// Returns the last error once `max_attempts` is exhausted.
pub fn execute_with_retry(policy: RetryPolicy) -> Result<Output, Error> {
    // ...
}
```

## Notes
- Every public kind needs its own angle: a struct explains what it represents, a field explains what it holds, an enum variant explains when it applies, a trait explains the contract implementors must uphold.
- `#![warn(missing_docs)]` at the crate root (or `missing_docs = "warn"` under `[workspace.lints.rust]`) catches an undocumented public item as a compiler warning, well before a reviewer would have to notice it by eye.
- A single doc comment is rarely enough on its own — pair it with an `# Examples` block (`doc-examples-section`) for anything non-trivial to call.

## References
- [doc-module-inner](doc-module-inner.md)
- [doc-examples-section](doc-examples-section.md)
