---
title: Pick the Map Type by Access Pattern, Not by Default Habit
impact: MEDIUM
impactDescription: Avoids sorting overhead you don't need or manual sorting you shouldn't have to do
tags: [collections, performance, api-design]
---

# Pick the Map Type by Access Pattern, Not by Default Habit [MEDIUM]

## Description
Three map types cover most needs, and each earns its keep through a different trade-off. `HashMap` gives up any notion of order in exchange for average O(1) lookup and insertion, which makes it the correct starting point whenever nothing downstream actually cares what order the entries come out in. `BTreeMap` accepts O(log n) operations in exchange for keeping its keys continuously sorted, which is the only way to support an efficient range query or a deterministic sorted iteration — capabilities a `HashMap` fundamentally cannot offer no matter how it's used. `IndexMap`, from the `indexmap` crate, occupies a third position: it keeps entries in the order they were first inserted while still offering roughly `HashMap`-speed lookup, which is exactly what something like config parsing wants, where the output needs to mirror the input's order rather than either a hash-determined order or a sorted one. Reaching for the wrong one means either paying for an ordering guarantee nothing downstream needs, or being stuck manually re-sorting output that a different map type would have given for free.

## Bad Example
```rust
use std::collections::HashMap;

fn word_counts(text: &str) -> HashMap<&str, usize> {
    let mut counts = HashMap::new();
    for word in text.split_whitespace() {
        *counts.entry(word).or_insert(0) += 1;
    }
    counts
    // Iterating this for a report produces a different order every run —
    // every caller that needs stable output has to sort it externally.
}
```

## Good Example
```rust
use std::collections::{BTreeMap, HashMap};
use indexmap::IndexMap;

// HashMap: default, fast, order irrelevant
fn total_scores<'a>(records: &[(&'a str, u32)]) -> HashMap<&'a str, u32> {
    let mut scores: HashMap<&'a str, u32> = HashMap::new();
    for &(name, score) in records {
        *scores.entry(name).or_insert(0) += score;
    }
    scores
}

// BTreeMap: sorted keys, range queries are only possible because keys stay sorted
fn events_in_range(log: &BTreeMap<u64, String>, start: u64, end: u64) -> Vec<(&u64, &String)> {
    log.range(start..=end).collect()
}

// IndexMap: insertion order preserved, O(1) average lookup — deterministic output
fn parse_config(pairs: &[(&str, &str)]) -> IndexMap<String, String> {
    pairs.iter().map(|(k, v)| (k.to_string(), v.to_string())).collect()
}
```

## Notes
- As a quick default: reach for `HashMap` when lookup speed is all that matters, `BTreeMap` when the code needs sorted iteration or a range query, `IndexMap` when output order has to match insertion order, and don't discount a plain `Vec` of tuples for a map with only a handful of entries, where the overhead of any hashing or tree structure outweighs the benefit.
- `HashMap`'s built-in hasher, SipHash-1-3, is deliberately resistant to an attacker crafting keys that all collide, which is valuable when keys come from untrusted input but is also slower than it needs to be for a purely internal cache never exposed to adversarial input — that's the specific case where swapping in a faster non-cryptographic hasher pays off.
- The entire reason `BTreeMap`'s O(log n) cost is worth paying is `.range()` and ordered iteration — if neither of those is actually needed by the code, there's no argument for choosing it over a `HashMap` "just in case" something later needs sorting.

## References
- [coll-seq-choice](coll-seq-choice.md)
- [coll-set-membership](coll-set-membership.md)
