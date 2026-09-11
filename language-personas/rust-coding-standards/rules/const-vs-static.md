---
title: Use const for Inlined Values, static for a Single Addressed Instance
impact: HIGH
impactDescription: Avoids unsound `static mut` globals and unintended value duplication across a binary
tags: [const, static, globals, safety]
---

# Use const for Inlined Values, static for a Single Addressed Instance [HIGH]

## Description
The distinction between `const` and `static` comes down to where the value physically lives. A `const` has no address of its own — every place it's referenced, the compiler substitutes a fresh copy of the value, which is cheap and appropriate for small things like a timeout number or a bitmask, but wasteful for anything large enough that duplicating it at every call site bloats the binary. A `static`, by contrast, is allocated exactly once and lives at a single fixed address for the program's entire run, which is what makes it the right choice for a sizeable lookup table or for anything that specifically needs to hand out a `&'static` reference. `static mut` is the one form worth avoiding outright: taking a reference to it is a deny-by-default lint under the 2024 edition — in practice a hard error unless someone deliberately silences it — because nothing stops two references from aliasing the same mutable memory without synchronization. When global state genuinely needs to change at runtime, atomics or one of `OnceLock`/`LazyLock` replace `static mut` without that hazard.

## Bad Example
```rust
// Large table declared `const` — risks duplication at every use site
const LOOKUP: [u8; 256] = [0u8; 256];

// `static mut` — referencing it triggers a deny-by-default lint in edition 2024
static mut COUNTER: u64 = 0;

fn increment() {
    unsafe {
        COUNTER += 1; // data race if called from multiple threads
    }
}
```

## Good Example
```rust
use std::sync::atomic::{AtomicU64, Ordering};
use std::sync::{LazyLock, OnceLock};

// Small values: `const` — inlined at each use, no address needed
const MAX_RETRIES: u32 = 3;
const TIMEOUT_MS: u64 = 5_000;

// Large data: `static` — one copy in the binary, shareable as `&'static`
static LOOKUP: [u8; 256] = [0u8; 256];

// Mutable global state — atomics instead of `static mut`
static REQUEST_COUNT: AtomicU64 = AtomicU64::new(0);

fn record_request() {
    REQUEST_COUNT.fetch_add(1, Ordering::Relaxed);
}

// Lazily initialized global — `LazyLock` (stable since 1.80)
static CONFIG_PATH: LazyLock<String> = LazyLock::new(|| {
    std::env::var("CONFIG_PATH").unwrap_or_else(|_| "/etc/app/config.toml".to_owned())
});

// Single-assignment global — `OnceLock`
static GREETING: OnceLock<String> = OnceLock::new();
```

## Notes
- As a rough default: small standalone values go in a `const`; a large table or anything that must produce a `&'static` reference goes in a `static`; a counter or flag that changes at runtime becomes a `static` holding an atomic type; a value computed once on first access is a `static LazyLock<T>`; a value set exactly once by whichever thread gets there first is a `static OnceLock<T>`.
- Because `const` is inlined wherever it's referenced, a large `const` value duplicates itself across every use site and can measurably inflate the binary — that duplication is precisely the cost `static` was designed to avoid.
- Editions before 2024 still compile `static mut` with only a warning, but every access still needs an `unsafe` block and the compiler still can't rule out a data race between two accesses — any existing occurrence is worth treating as technical debt to migrate off, not a pattern to reach for in new code.

## References
- [own-mutex-interior](own-mutex-interior.md)
- [unsafe-minimize-scope](unsafe-minimize-scope.md)
