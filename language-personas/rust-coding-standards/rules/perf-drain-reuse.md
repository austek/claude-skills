---
title: Use drain() to Move Elements Out Without Losing the Allocation
impact: MEDIUM
impactDescription: Replaces a per-iteration allocate/free cycle with one allocation reused across the loop
tags: [performance, collections, allocation, drain]
---

# Use drain() to Move Elements Out Without Losing the Allocation [MEDIUM]

## Description
`Vec::drain(range)` removes elements and hands them back through an iterator, but unlike `into_iter()` it doesn't consume the `Vec` itself — the underlying allocation survives, ready to be filled again. That makes it the right tool for a loop that repeatedly needs an empty container with the same rough capacity: allocating a fresh `Vec` on every pass and letting the old one drop is a full allocate-then-free cycle every iteration, while `drain(..)` followed by reuse (or simply `.clear()`, which discards elements the same way but keeps capacity without needing to iterate them) pays that allocation cost exactly once.

## Bad Example
```rust
fn reuse_buffer(iterations: usize) {
    for _ in 0..iterations {
        let mut buffer = Vec::new(); // new allocation every pass
        fill_buffer(&mut buffer);
        process(&buffer);
        // buffer drops here, freeing the allocation just made
    }
}
```

## Good Example
```rust
fn reuse_buffer(iterations: usize) {
    let mut buffer = Vec::new();
    for _ in 0..iterations {
        buffer.clear(); // keeps the allocated capacity
        fill_buffer(&mut buffer);
        process(&buffer);
    }
}

fn transfer_all(src: &mut Vec<Item>, dst: &mut Vec<Item>) {
    dst.extend(src.drain(..)); // moves elements; src keeps its capacity, now empty
}
```

## Notes
- `.clear()` and `.drain(..)` both keep capacity and remove all elements — the difference is that `drain` hands back an iterator over what was removed, useful when those elements need to go somewhere else, while `clear` just drops them in place.
- `std::mem::take(&mut vec)` is a different tool for a different job: it swaps in a fresh, zero-capacity `Vec` and returns the original by value — reach for it when ownership of the whole collection needs to move out, not when the goal is reusing the same buffer.
- `HashMap::drain()` and `HashSet::drain()` follow the identical pattern for map and set types — draining a map's entries for processing leaves the map empty but still holding its bucket allocation.

## References
- [mem-reuse-collections](mem-reuse-collections.md)
- [perf-extend-batch](perf-extend-batch.md)
- [mem-with-capacity](mem-with-capacity.md)
