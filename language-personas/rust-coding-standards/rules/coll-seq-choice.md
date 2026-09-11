---
title: Default to Vec, Reach for VecDeque for Queue Behavior
impact: MEDIUM
impactDescription: Turns O(n) front removal into O(1) amortized, avoiding quadratic queue processing
tags: [collections, performance, vec]
---

# Default to Vec, Reach for VecDeque for Queue Behavior [MEDIUM]

## Description
`Vec<T>` stores its elements contiguously in memory, which is exactly the layout that makes a modern CPU's prefetcher effective — reading element N makes it likely element N+1 is already in cache by the time it's needed, so `Vec` should be the automatic choice for a growable sequence unless something specific rules it out. `VecDeque<T>` is built as a ring buffer instead, which costs it nothing for ordinary indexed access but buys O(1) amortized insertion and removal at *both* ends rather than just the back — exactly the shape needed for FIFO queue processing or a fixed-size sliding window. `LinkedList<T>` gives up contiguous storage entirely, allocating each element as its own heap node and chasing pointers between them on every traversal, which in measured practice tends to lose to `Vec` or `VecDeque` even in the scenarios it was theoretically designed for — enough so that the standard library's own documentation steers people away from it absent a profiled reason to reach for it.

## Bad Example
```rust
fn process_queue(items: Vec<String>) {
    let mut queue = items;
    while !queue.is_empty() {
        // O(n): every remaining element shifts left after removal —
        // a loop of n items becomes O(n²)
        let item = queue.remove(0);
        println!("processing: {item}");
    }
}
```

## Good Example
```rust
use std::collections::VecDeque;

fn process_queue(items: impl IntoIterator<Item = String>) {
    // O(1) pop_front — the right tool for a FIFO queue
    let mut queue: VecDeque<String> = items.into_iter().collect();
    while let Some(item) = queue.pop_front() {
        println!("processing: {item}");
    }
}

fn sliding_window_max(values: &[i32], k: usize) -> Vec<i32> {
    // VecDeque also serves as a fixed-size sliding window
    let mut window: VecDeque<i32> = VecDeque::with_capacity(k);
    let mut result = Vec::with_capacity(values.len().saturating_sub(k) + 1);
    for &v in values {
        window.push_back(v);
        if window.len() > k {
            window.pop_front();
        }
        if window.len() == k {
            result.push(*window.iter().max().unwrap());
        }
    }
    result
}
```

## Notes
- The practical decision tree is short: default to `Vec<T>`, switch to `VecDeque<T>` the moment the access pattern is a queue, deque, or fixed-size window, and treat `LinkedList<T>` as essentially off the table unless a specific profiling result says otherwise for a specific workload.
- Committing to `Vec` early isn't a risky bet, since converting between it and `VecDeque` later via `From`/`Into` is cheap — if the access pattern turns out to need deque behavior after all, switching representations doesn't mean rewriting the surrounding code from scratch.
- Removing an item from the middle of a `VecDeque` is still O(n), same as a `Vec` — so a workload that genuinely needs fast removal at arbitrary positions calls for reconsidering the data structure entirely (an index-based structure, a different algorithm), not for defaulting to `LinkedList`, which doesn't actually solve that problem efficiently either.

## References
- [coll-binaryheap](coll-binaryheap.md)
- [mem-with-capacity](mem-with-capacity.md)
