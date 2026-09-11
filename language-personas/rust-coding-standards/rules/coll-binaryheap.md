---
title: Use BinaryHeap for a Priority Queue or Repeated Max-Extraction
impact: MEDIUM
impactDescription: Turns O(n) per extraction into O(log n), avoiding quadratic blowup over many pops
tags: [collections, performance, priority-queue]
---

# Use BinaryHeap for a Priority Queue or Repeated Max-Extraction [MEDIUM]

## Description
`std::collections::BinaryHeap<T>` maintains its elements so the largest one is always reachable in O(1) via `peek`, and both inserting a new element and removing the current maximum cost O(log n) rather than requiring a full pass over the data. That matters for any workload that keeps adding and removing "the biggest item so far" repeatedly, like a task scheduler picking the next job to run — the naive alternative of scanning a plain `Vec` for its maximum and removing it costs O(n) per extraction, which compounds into O(n²) across a full run, while the heap keeps the same sequence of operations down around O(n log n). Nothing about the heap itself is tied to "biggest" specifically — wrapping each value in `std::cmp::Reverse<T>` inverts the comparison the heap uses internally, which is the standard trick for getting min-first behavior out of what's structurally still a max-heap.

## Bad Example
```rust
fn top_priority_task(tasks: &mut Vec<(u32, String)>) -> Option<String> {
    if tasks.is_empty() {
        return None;
    }
    // O(n) scan for the max, then O(n) shift to remove it — O(n) per call
    let max_idx = tasks.iter().enumerate().max_by_key(|(_, (p, _))| *p).map(|(i, _)| i)?;
    Some(tasks.remove(max_idx).1)
}
// Repeated calls in a loop become O(n²) overall.
```

## Good Example
```rust
use std::cmp::Reverse;
use std::collections::BinaryHeap;

#[derive(Eq, PartialEq, Ord, PartialOrd)]
struct Task {
    priority: u32, // higher = more urgent
    name: String,
}

fn run_scheduler(tasks: impl IntoIterator<Item = (u32, &'static str)>) {
    // O(log n) push and pop — max-priority task extracted first
    let mut heap: BinaryHeap<Task> = tasks
        .into_iter()
        .map(|(priority, name)| Task { priority, name: name.to_string() })
        .collect();

    while let Some(task) = heap.pop() {
        println!("running [priority={}]: {}", task.priority, task.name);
    }
}

fn top_k_largest(values: &[i32], k: usize) -> Vec<i32> {
    // Min-heap of size k, via Reverse<i32> — keeps only the k largest elements seen so far
    let mut min_heap: BinaryHeap<Reverse<i32>> = BinaryHeap::with_capacity(k + 1);
    for &v in values {
        min_heap.push(Reverse(v));
        if min_heap.len() > k {
            min_heap.pop(); // discard the current minimum
        }
    }
    let mut result: Vec<i32> = min_heap.into_iter().map(|Reverse(v)| v).collect();
    result.sort_unstable_by(|a, b| b.cmp(a));
    result
}
```

## Notes
- `.peek()` looks at the current maximum without touching the heap's structure, in constant time; `.into_sorted_vec()` consumes the whole heap and hands back everything in sorted order, which costs O(n log n) since it's effectively sorting on the way out.
- What a `BinaryHeap` is structurally bad at is touching an element that isn't currently the max — there's no efficient way to remove an arbitrary entry or adjust its priority once it's inside the heap. A workload that genuinely needs that should look at a keyed priority-queue crate, or restructure the problem around inserting a fresh replacement entry and simply ignoring stale ones as they surface from `pop`.
- The heap orders elements purely by whatever `Ord` the element type implements, so for a custom struct, the derived or hand-written `Ord` impl *is* the definition of "biggest" as far as the heap is concerned — it's worth double-checking that comparison logic actually reflects the intended priority before trusting the pop order.

## References
- [coll-seq-choice](coll-seq-choice.md)
