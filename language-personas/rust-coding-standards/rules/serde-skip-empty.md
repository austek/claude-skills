---
title: Omit Empty Fields with serde skip_serializing_if
impact: LOW
impactDescription: Trims null/empty noise from serialized payloads and logs
tags: [serde, serialization, payload-size]
---

# Omit Empty Fields with serde skip_serializing_if [LOW]

## Description
Left to its defaults, serde writes out every field regardless of whether it's actually carrying information — an `Option<T>` that's `None` becomes a literal `null` in the output, and an empty `Vec` becomes `[]`. Multiply that across a struct with several optional fields and the payload fills up with noise that adds nothing, and in some client implementations a present-but-null key is treated differently than an absent one, which can introduce a subtle mismatch. `#[serde(skip_serializing_if = "predicate")]` lets a field opt out of appearing in the output whenever some function of its value returns true, so an empty or absent value is simply omitted rather than written out as `null` or `[]`. `#[serde(skip)]` is the more absolute version of the same idea — the field is invisible to serde in both directions, which fits fields that exist for internal bookkeeping and were never meant to leave the process in the first place.

## Bad Example
```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct ApiResponse {
    id: u64,
    name: String,
    description: Option<String>,  // serializes as null when None
    tags: Vec<String>,            // serializes as [] when empty
}
// Produces: {"id":1,"name":"Alice","description":null,"tags":[]}
```

## Good Example
```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug, Default)]
struct ApiResponse {
    id: u64,
    name: String,
    #[serde(skip_serializing_if = "Option::is_none")]
    description: Option<String>,
    #[serde(skip_serializing_if = "Vec::is_empty")]
    tags: Vec<String>,
    // Internal field, excluded entirely from the wire format
    #[serde(skip)]
    _cache_key: Option<String>,
}
// Produces: {"id":1,"name":"Alice"} — absent fields are simply omitted
```

## Notes
- The predicate given to `skip_serializing_if` just needs to resolve to a `fn(&T) -> bool` — the common cases are `Option::is_none`, `Vec::is_empty`, and `String::is_empty`, but nothing stops it from pointing at a hand-written function for a more specific condition.
- Because `#[serde(skip)]` removes a field from both serialization and deserialization, the struct needs a `Default` implementation so deserialization has something to put there when the field never appears in the input at all. The narrower `#[serde(skip_serializing)]` and `#[serde(skip_deserializing)]` cover just one direction, which fits a field that's still read from legacy input but should never be written out again going forward.
- `skip_serializing_if` on its own only changes serialization — pairing it with `#[serde(default)]` on the same field makes deserialization tolerate the key being absent too, instead of treating the now-omittable field as still required on input.

## References
- [serde-default-compat](serde-default-compat.md)
- [serde-rename-all](serde-rename-all.md)
