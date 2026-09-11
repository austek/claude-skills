---
title: Inline Shared Fields with serde flatten
impact: MEDIUM
impactDescription: Removes duplicated field groups across DTOs while keeping the wire format flat
tags: [serde, serialization, api-design, dto]
---

# Inline Shared Fields with serde flatten [MEDIUM]

## Description
A handful of fields tend to recur across otherwise-unrelated API response types — pagination counters, audit timestamps, a standard error envelope. Copying those fields into every struct that needs them works at first, but it's the kind of duplication that rots: a field gets added to the shared concept later, and it's easy to update three of the four structs that needed it and miss the fourth. `#[serde(flatten)]` solves this by letting one struct's fields be spliced directly into another's wire representation during (de)serialization, so the Rust side can factor the shared fields into their own type while the JSON on the wire still looks flat, with no nested `"pagination": {...}` wrapper visible to consumers. The same attribute, aimed at a `HashMap` field instead of a named struct, works in the opposite direction too — it can soak up every key the rest of the struct doesn't claim, which is a convenient way to keep extra or dynamic metadata around without declaring a field for each one in advance.

## Bad Example
```rust
use serde::{Serialize, Deserialize};

// Pagination fields copy-pasted into every list response
#[derive(Serialize, Deserialize, Debug)]
struct UserListResponse {
    users: Vec<String>,
    page: u32,
    per_page: u32,
    total: u64,
}

#[derive(Serialize, Deserialize, Debug)]
struct PostListResponse {
    posts: Vec<String>,
    page: u32,
    per_page: u32,
    total: u64,
}
```

## Good Example
```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct Pagination {
    page: u32,
    per_page: u32,
    total: u64,
}

#[derive(Serialize, Deserialize, Debug)]
struct UserListResponse {
    users: Vec<String>,
    #[serde(flatten)]
    pagination: Pagination,
}

#[derive(Serialize, Deserialize, Debug)]
struct PostListResponse {
    posts: Vec<String>,
    #[serde(flatten)]
    pagination: Pagination,
}
// UserListResponse serializes to {"users":[...],"page":1,"per_page":20,"total":100} —
// Pagination's fields appear at the top level, not nested under "pagination".
```

## Notes
- Both a nested struct and a map type can be the target of `#[serde(flatten)]`, as long as whatever's flattened implements `Serialize`/`Deserialize` itself; a struct is even free to flatten more than one field this way, provided their key sets don't collide once merged.
- Reaching for `HashMap<String, serde_json::Value>` as the flattened field turns it into a general catch-all bucket for whatever keys aren't claimed by named fields — a common pattern for config structs and proxies that need to pass unknown metadata through untouched.
- This attribute and `#[serde(deny_unknown_fields)]` cannot both apply to the same struct: `flatten` depends on being able to forward keys that don't match a named field, and `deny_unknown_fields` treats exactly those keys as errors before `flatten` gets a chance to claim them. `flatten` also routes through a slower, buffered deserialization path internally, so it's worth avoiding on a genuinely hot path, and it has no equivalent in a non-self-describing binary format like `bincode`, which has no concept of an arbitrary-key map to merge into.

## References
- [serde-deny-unknown-fields](serde-deny-unknown-fields.md)
- [serde-enum-representation](serde-enum-representation.md)
