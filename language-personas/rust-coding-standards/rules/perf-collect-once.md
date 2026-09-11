---
title: Chain Iterator Adapters and Collect Exactly Once
impact: MEDIUM
impactDescription: Turns N allocations and N passes over the data into one of each
tags: [performance, iterators, allocation, collections]
---

# Chain Iterator Adapters and Collect Exactly Once [MEDIUM]

## Description
Every call to `.collect()` walks its source iterator to completion and allocates a new collection to hold the result — calling it two or three times in a row, once per pipeline stage, means the same data gets copied into a new allocation at every stage instead of flowing through a single pass. Rust's iterator adapters are built to be chained: `.filter().filter().map()` composes into one lazy pipeline that a single trailing `.collect()` drives to completion in one pass, with one allocation. The same mistake shows up in miniature when a chain is collected purely to call `.len()` on the result — `.count()` gets the same number by consuming the iterator without ever materializing a collection.

## Bad Example
```rust
fn process_users(users: Vec<User>) -> Vec<String> {
    let active: Vec<_> = users.into_iter().filter(|u| u.is_active).collect();
    let verified: Vec<_> = active.into_iter().filter(|u| u.is_verified).collect();
    verified.into_iter().map(|u| u.name).collect() // three allocations, three passes
}
```

## Good Example
```rust
fn process_users(users: Vec<User>) -> Vec<String> {
    users
        .into_iter()
        .filter(|u| u.is_active)
        .filter(|u| u.is_verified)
        .map(|u| u.name)
        .collect() // one allocation, one pass
}

fn count_valid(items: &[Item]) -> usize {
    items.iter().filter(|i| i.is_valid()).count() // no allocation at all
}
```

## Notes
- An intermediate collection is legitimate, not a mistake, when the data genuinely needs multiple passes — sorting requires a concrete buffer to sort in place, and code that reads `.len()` and iterates the same data twice needs it materialized once.
- When collecting is unavoidable, pairing it with a known-size `Vec::with_capacity` (or `.extend()` into a pre-sized buffer) avoids the reallocation-as-it-grows cost on top of the allocation itself.
- `impl Iterator<Item = T>` as a return type lets a function hand back a lazy pipeline without forcing the caller to decide when to collect — deferring that decision to whoever actually consumes the result.

## References
- [perf-iter-lazy](perf-iter-lazy.md)
- [mem-with-capacity](mem-with-capacity.md)
- [anti-collect-intermediate](anti-collect-intermediate.md)
