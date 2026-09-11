---
title: Reserve the to_ Prefix for Conversions That Cost Something
impact: MEDIUM
impactDescription: Warns callers before they call it in a loop and pay the allocation repeatedly
tags: [naming, api-design, conversions, allocation]
---

# Reserve the to_ Prefix for Conversions That Cost Something [MEDIUM]

## Description
`to_` is the middle tier of Rust's conversion-naming triad: it marks a method that produces a new, independently owned value from a borrow — `&Self -> U` — and does so by allocating, cloning, or otherwise doing real work. `String::to_uppercase()`, `[T]::to_vec()`, and `str::to_owned()` are all `to_` because each one has to create fresh memory to satisfy its return type. The naming matters because it's the caller's only signal, short of reading the source, that a value returned from this call isn't free to request repeatedly — a `to_` method inside a hot loop is worth a second look, an `as_` method inside the same loop is not.

## Bad Example
```rust
struct Slug(String);

impl Slug {
    // as_ implies free, but building a new String here is not free.
    fn as_display(&self) -> String {
        self.0.replace('-', " ")
    }
}
```

## Good Example
```rust
struct Slug(String);

impl Slug {
    // to_ tells the caller: this allocates, consider caching the result.
    fn to_display(&self) -> String {
        self.0.replace('-', " ")
    }

    // as_ is honest here: no allocation, just a reference.
    fn as_str(&self) -> &str {
        &self.0
    }
}
```

## Notes
- `ToOwned::to_owned()` generalizes the same convention across the standard library: `&str::to_owned() -> String`, `&[T]::to_owned() -> Vec<T>` — both allocate to produce an owned counterpart of borrowed data.
- The triad's third leg, `into_`, differs from `to_` in receiver: `into_` consumes `self` and is usually cheap (a move), while `to_` borrows `&self` and is usually expensive (a copy) — see `name-into-ownership`.
- If a `to_` method's cost is more than "allocates a value roughly proportional to the input" — say, it does network I/O or heavy computation — consider whether the method belongs in this naming family at all, or needs a more explicit name (`compute_hash`, `fetch_remote`) that doesn't hide behind a generic conversion prefix.

## References
- [name-as-free](name-as-free.md)
- [name-into-ownership](name-into-ownership.md)
