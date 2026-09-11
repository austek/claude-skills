---
title: Prefer thread_local! Over static mut
impact: CRITICAL
impactDescription: Removes undefined behavior from cross-thread access to global state
tags: [concurrency, thread-local, static-mut, safety]
---

# Prefer thread_local! Over static mut [CRITICAL]

## Description
`static mut` requires `unsafe` at every access and is undefined behavior the moment two threads touch it concurrently — the compiler has no way to rule that out. As of the 2024 edition, taking any reference to a `static mut` is a hard compile error (`static_mut_refs`). `thread_local!` gives each thread its own independent copy of the value instead, reached through safe APIs via `Cell` for `Copy` types or `RefCell` for everything else, with no synchronization overhead and no `unsafe` block anywhere in the call path.

## Bad Example
```rust
// 2024 edition: referencing static mut is a hard error (static_mut_refs lint)
static mut BUFFER: Vec<u8> = Vec::new();

fn append_to_buffer(data: &[u8]) {
    // UB if called concurrently from multiple threads
    unsafe {
        BUFFER.extend_from_slice(data);
    }
}
```

## Good Example
```rust
use std::cell::RefCell;

thread_local! {
    static BUFFER: RefCell<Vec<u8>> = RefCell::new(Vec::with_capacity(4096));
}

fn append_to_buffer(data: &[u8]) {
    BUFFER.with_borrow_mut(|buf| buf.extend_from_slice(data));
}

fn flush_buffer() -> Vec<u8> {
    BUFFER.with_borrow_mut(|buf| std::mem::take(buf))
}
```

## Notes
- `with_borrow` / `with_borrow_mut` (stable since 1.73) are the concise form on `LocalKey<RefCell<T>>` — prefer them over `with(|v| v.borrow_mut())`.
- For `Copy` types, `Cell` avoids borrow-check overhead entirely: `thread_local! { static N: Cell<u32> = Cell::new(0); }`.
- Thread-local destructors run automatically when the owning thread exits.
- A thread-local value is strictly per-thread and invisible to other threads; for read-only global state shared across threads, use an ordinary immutable `static` instead.

## References
- [own-refcell-interior](own-refcell-interior.md)
- [own-mutex-interior](own-mutex-interior.md)
