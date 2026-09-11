---
title: Use HashSet or BTreeSet for Membership Tests, Not Linear Vec::contains
impact: HIGH
impactDescription: Turns an O(n * m) membership check loop into O(n + m)
tags: [collections, performance, algorithms]
---

# Use HashSet or BTreeSet for Membership Tests, Not Linear Vec::contains [HIGH]

## Description
Checking whether a `Vec` contains a given value means walking it from the front until a match turns up or the end is reached — O(n) in the worst case, every single call. That cost is easy to miss in isolation but compounds fast inside a loop: testing m different items against an n-element `Vec` means paying that O(n) scan m separate times, for a total of O(n × m), which grows quadratically as both sides of the problem grow. A `HashSet` restructures the same data so each membership check is O(1) on average instead, which turns that same loop into O(n + m) overall — a difference that becomes very noticeable once n and m are more than trivially small. Deduplicating a list follows the same underlying issue: checking "have I seen this already?" against a growing `Vec` is the identical O(n)-per-check pattern, while checking it against a `HashSet` is one `insert` call that reports whether the value was already present. None of this is a reason to abandon `Vec` for tiny collections — somewhere around a handful of elements, the overhead of computing a hash exceeds what a linear scan would have cost anyway, and if preserving duplicates or the original order is actually the goal, `Vec` remains the right tool regardless of size.

## Bad Example
```rust
fn find_common(all_users: &[String], active_ids: &[String]) -> Vec<String> {
    let mut common = Vec::new();
    for user in all_users {
        // O(n) per iteration → O(n * m) total
        if active_ids.contains(user) {
            common.push(user.clone());
        }
    }
    common
}

fn deduplicate(items: Vec<String>) -> Vec<String> {
    let mut seen: Vec<String> = Vec::new();
    for item in items {
        if !seen.contains(&item) { // O(n) per item — quadratic overall
            seen.push(item);
        }
    }
    seen
}
```

## Good Example
```rust
use std::collections::HashSet;

// O(n + m): build the set once, then test each user in O(1)
fn find_common(all_users: &[String], active_ids: &[String]) -> Vec<String> {
    let active: HashSet<&String> = active_ids.iter().collect();
    all_users.iter().filter(|u| active.contains(u)).cloned().collect()
}

// Dedup while preserving insertion order: track seen items in a HashSet
fn deduplicate_ordered(items: Vec<String>) -> Vec<String> {
    let mut seen = HashSet::with_capacity(items.len());
    items.into_iter().filter(|s| seen.insert(s.clone())).collect()
}
```

## Notes
- Default to `HashSet<T>` whenever the goal is just a fast yes/no membership check and iteration order is irrelevant; move to `BTreeSet<T>` the moment sorted iteration or range queries enter the picture; and don't rule out a `Vec<T>` with `.contains` for a genuinely small set, or one where duplicates or insertion order are part of the actual requirement.
- Building a `HashSet` up front from a source whose size is already known is a good moment to call `HashSet::with_capacity(n)` — it sidesteps the repeated rehashing the set would otherwise do as it grows organically one insertion at a time.
- The one-pass dedup idiom `.filter(|s| seen.insert(s.clone()))` works because `HashSet::insert` itself reports whether the value was already present, returning `false` in that case — that's what lets a single filter step replace what would otherwise be a separate contains-check followed by an insert.

## References
- [coll-map-choice](coll-map-choice.md)
- [mem-with-capacity](mem-with-capacity.md)
