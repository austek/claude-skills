---
title: Spell Out the Contract in a # Safety Section on Every unsafe fn
impact: CRITICAL
impactDescription: Missing safety contract means callers can't avoid triggering undefined behavior
tags: [documentation, unsafe, safety, rustdoc]
---

# Spell Out the Contract in a # Safety Section on Every unsafe fn [CRITICAL]

## Description
Marking a function `unsafe` tells the compiler to stop checking a set of invariants and trust the caller to uphold them instead — but the compiler can't tell the caller what those invariants actually are. That's the entire job of a `# Safety` section: without it, calling the function correctly requires either reading the implementation closely enough to reverse-engineer the preconditions, or guessing. A guess that's wrong isn't a compile error or even necessarily a panic — it's undefined behavior, which can manifest as anything from a wrong value to memory corruption, often far away from the call site that triggered it. This is why `# Safety` isn't a nice-to-have the way most doc sections are; an `unsafe fn` without one is, in a real sense, unusable safely by anyone who isn't its author.

## Bad Example
```rust
/// Creates a `String` from raw parts.
pub unsafe fn string_from_raw(ptr: *mut u8, len: usize, cap: usize) -> String {
    // caller has no way to know what ptr/len/cap must satisfy
    String::from_raw_parts(ptr, len, cap)
}
```

## Good Example
```rust
/// Creates a `String` from raw parts.
///
/// # Safety
///
/// The caller must guarantee that:
/// - `ptr` was allocated by the same allocator `String` uses
/// - `len <= cap`, and `cap` matches the allocation `ptr` was given
/// - the first `len` bytes at `ptr` are valid UTF-8
/// - ownership of the allocation transfers to the returned `String` —
///   no other code may use `ptr` after this call
pub unsafe fn string_from_raw(ptr: *mut u8, len: usize, cap: usize) -> String {
    String::from_raw_parts(ptr, len, cap)
}
```

## Notes
- An `unsafe trait` needs the same treatment on the trait definition itself — the `# Safety` section there documents what any `unsafe impl` of it must guarantee to be sound, not what a caller of its methods must do.
- A safe function that contains an internal `unsafe` block still needs a `// SAFETY:` comment at that block explaining why the invariant holds locally — `unsafe-safety-comment` covers that narrower, code-level case; `# Safety` in the doc comment is the public-facing contract for an `unsafe fn` itself.
- Cover pointer validity, alignment, initialization, aliasing, and any size/lifetime bound explicitly — a vague "caller must ensure this is used correctly" gives the reader nothing they didn't already know from the `unsafe` keyword itself.

## References
- [unsafe-safety-comment](unsafe-safety-comment.md)
- [doc-panics-section](doc-panics-section.md)
