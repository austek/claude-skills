---
title: Require SAFETY Comments on unsafe Blocks
impact: HIGH
impactDescription: Forces the invariant an unsafe block relies on to be written down, not just held in the author's head
tags: [tooling, clippy, unsafe, safety, documentation]
---

# Require SAFETY Comments on unsafe Blocks [HIGH]

## Description
An `unsafe` block is a promise to the compiler that invariants it can't check itself — pointer validity, absence of aliasing, correct alignment — are upheld by the surrounding code. Without a comment recording what that promise actually is, the only record of it lives in whatever the author was thinking at the time, which is gone the moment they move to a different task, and completely invisible to the next person who has to change nearby code without breaking that invariant. Clippy's `undocumented_unsafe_blocks` lint forces the invariant into text: every `unsafe { .. }` must be immediately preceded by a `// SAFETY:` comment, which converts an implicit assumption into something a reviewer can actually check and a future change can be measured against.

## Bad Example
```rust
pub fn read_data(ptr: *const u8, len: usize) -> &[u8] {
    unsafe {
        std::slice::from_raw_parts(ptr, len) // no record of why this is sound
    }
}
```

## Good Example
```rust
pub fn read_data(ptr: *const u8, len: usize) -> &[u8] {
    // SAFETY: caller guarantees `ptr` is valid for reads of `len` bytes,
    // is properly aligned for u8, points to initialized memory, and that
    // no mutable reference to this memory exists for the returned lifetime.
    unsafe { std::slice::from_raw_parts(ptr, len) }
}
```

## Notes
- A useful SAFETY comment covers three things: which preconditions the unsafe operation actually depends on, why those preconditions hold at this call site (a prior check, a type-level guarantee, a caller contract), and what breaks if they don't — a comment that just restates "this is safe" without any of that adds no real information.
- `unsafe impl Send for MyType {}` / `unsafe impl Sync for MyType {}` need the same treatment as a raw-pointer dereference — the SAFETY comment should explain what about the type's internals makes sending or sharing it across threads actually sound (no non-atomic interior mutability, no raw pointers to thread-local data, etc).
- Pair `undocumented_unsafe_blocks` with `multiple_unsafe_ops_per_block`, which pushes toward one unsafe operation per block — a SAFETY comment covering three different unchecked operations at once is much easier to get subtly wrong than three comments each covering one.
- This lint checks that a comment exists in the right place; it can't verify the comment's claims are actually true. Treat every SAFETY comment as something a reviewer needs to independently reason about, not as proof the code is correct just because it's documented.

## References
- [doc-safety-section](../../rust-coding-standards/rules/doc-safety-section.md)
- [lint-deny-correctness](lint-deny-correctness.md)
- [type-repr-transparent](../../rust-coding-standards/rules/type-repr-transparent.md)
