---
title: Use the Entry API Instead of Check-Then-Insert on a Map
impact: MEDIUM
impactDescription: Collapses two hash lookups into one for every insert-or-update
tags: [performance, hashmap, entry-api, collections]
---

# Use the Entry API Instead of Check-Then-Insert on a Map [MEDIUM]

## Description
`if map.contains_key(&key) { ... } else { map.insert(key, ...) }` hashes the key and walks the bucket twice — once for `contains_key`, once again for whichever of `get_mut`/`insert` runs next. `HashMap::entry` (and the identical API on `BTreeMap`) does the lookup exactly once, returning a handle that's either `Occupied` or `Vacant` and can be updated or filled in place from there. Beyond the speed, it's also the more direct way to say what the code actually means — "look this key up, and either modify what's there or insert a default" is one operation conceptually, and the entry API makes it one operation in code too.

## Bad Example
```rust
use std::collections::HashMap;

fn increment(map: &mut HashMap<String, u32>, key: String) {
    if map.contains_key(&key) {         // lookup #1
        *map.get_mut(&key).unwrap() += 1; // lookup #2
    } else {
        map.insert(key, 1);              // lookup #3 (on the insert path)
    }
}
```

## Good Example
```rust
use std::collections::HashMap;

fn increment(map: &mut HashMap<String, u32>, key: String) {
    *map.entry(key).or_insert(0) += 1; // single lookup either way
}

fn word_count(text: &str) -> HashMap<&str, usize> {
    let mut counts = HashMap::new();
    for word in text.split_whitespace() {
        *counts.entry(word).or_insert(0) += 1;
    }
    counts
}
```

## Notes
- `.or_insert_with(f)` defers constructing the default value until it's actually needed (lazy), while `.or_insert(val)` always constructs `val` up front even on the occupied path — prefer `_with` when the default is non-trivial to build (a `Vec::new()` is free either way, but a `Config::default()` reading files is not).
- `.and_modify(f).or_insert_with(g)` handles "update if present, otherwise insert a default" in one expression — the closure passed to `and_modify` only runs on the occupied branch.
- For anything beyond a simple insert-or-update, matching on `Entry::Occupied`/`Entry::Vacant` directly exposes the full API (`entry.get_mut()`, `entry.insert()`, `entry.remove()`) for more elaborate insert-or-update logic without giving up the single-lookup property.

## References
- [perf-extend-batch](perf-extend-batch.md)
- [mem-with-capacity](mem-with-capacity.md)
- [coll-map-choice](coll-map-choice.md)
