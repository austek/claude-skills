---
title: Batch Insertions with extend() Instead of Repeated push()
impact: MEDIUM
impactDescription: Lets the collection size itself from the iterator instead of growing by doubling
tags: [performance, collections, allocation, iterators]
---

# Batch Insertions with extend() Instead of Repeated push() [MEDIUM]

## Description
A loop of individual `.push()` calls grows a `Vec` the only way it can without foreknowledge of the final size: by doubling capacity whenever it runs out, which means a handful of reallocate-and-copy events scattered across the loop. `.extend(iterator)` sees the whole batch at once — when the iterator reports an accurate `size_hint()`, `extend` reserves the needed capacity up front and fills it in a single pass, cutting that handful of reallocations down to at most one. The same idea applies to combining two existing collections: `a.extend(b)` (or `a.append(&mut b)` for a `Vec`, which additionally leaves `b` empty rather than consuming it) is both clearer and faster than looping over `b` and pushing each element into `a`.

## Bad Example
```rust
fn build_list(source: impl Iterator<Item = i32>) -> Vec<i32> {
    let mut list = Vec::new();
    for x in source {
        list.push(x); // capacity doubles repeatedly as this grows
    }
    list
}
```

## Good Example
```rust
fn build_list(source: impl Iterator<Item = i32>) -> Vec<i32> {
    source.collect() // or: let mut list = Vec::new(); list.extend(source);
}

fn merge_all(chunks: Vec<Vec<Item>>) -> Vec<Item> {
    let total: usize = chunks.iter().map(Vec::len).sum();
    let mut result = Vec::with_capacity(total); // reserve once, up front
    for chunk in chunks {
        result.extend(chunk);
    }
    result
}
```

## Notes
- `extend_from_slice(&[T])` is the specialized form for `Copy` element types coming from a slice — it can use a raw memory copy instead of iterating element by element, faster than the generic `extend` for that specific case.
- Pairing `Vec::with_capacity(total)` with `extend` gets the best case even when the source iterator's `size_hint()` is inaccurate or absent (as with some `flat_map`/`filter` chains) — the reservation happens explicitly instead of relying on the iterator to report it.
- `String` benefits from the identical reasoning: building one from many `&str` pieces via repeated `push_str` risks several reallocations, while `String::with_capacity(total_len)` up front, or simply `parts.concat()`/`parts.join("")`, does the sizing once.

## References
- [mem-with-capacity](mem-with-capacity.md)
- [perf-drain-reuse](perf-drain-reuse.md)
- [mem-reuse-collections](mem-reuse-collections.md)
