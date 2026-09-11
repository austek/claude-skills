---
title: Use serde default for Optional and Backward-Compatible Fields
impact: HIGH
impactDescription: Prevents old payloads from breaking deserialization when new fields are added
tags: [serde, serialization, compatibility]
---

# Use serde default for Optional and Backward-Compatible Fields [HIGH]

## Description
Serde's default behavior is strict: if a key a struct expects isn't present in the payload being parsed, deserialization stops with a missing-field error rather than guessing. That's reasonable the first time a struct is written, but it becomes a liability the moment a new field is added later — every payload produced before that change now fails to parse, even though nothing about the older data was actually wrong. `#[serde(default)]`, applied either to a single field or to the whole struct, changes the failure mode: instead of erroring, serde reaches for the type's `Default` implementation (or a named fallback function) to fill in whatever key didn't show up, so structs can grow without retroactively breaking every payload that predates the new field.

## Bad Example
```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct Config {
    host: String,
    port: u16,
    timeout_secs: u64,  // newly added — old configs lack this field and fail to parse
    retries: u32,        // newly added — same problem
}
```

## Good Example
```rust
use serde::{Serialize, Deserialize};

fn default_retries() -> u32 { 3 }

#[derive(Serialize, Deserialize, Debug)]
struct Config {
    host: String,
    port: u16,
    // Fills from Default::default() (0u64) if missing
    #[serde(default)]
    timeout_secs: u64,
    // Fills from the named function if the bare Default would be wrong
    #[serde(default = "default_retries")]
    retries: u32,
}

// Container-level default: every field falls back to its Default if absent
#[derive(Serialize, Deserialize, Debug, Default)]
#[serde(default)]
struct FeatureFlags {
    enable_caching: bool,
    enable_metrics: bool,
    max_connections: u32,
}
```

## Notes
- Putting `#[serde(default)]` on one field only relaxes that field — every other field in the struct is still mandatory unless it also carries the attribute. When `Default::default()` wouldn't give a sensible fallback (zero is rarely the right default retry count), point at a named function instead with `#[serde(default = "path")]`.
- Putting the attribute on the container relaxes every field at once and requires the struct itself to implement `Default`. That's convenient for something like an all-optional feature-flag struct, but it comes with a real cost: a typo in a field name no longer produces an error, it just silently falls back to the default. Reserve the container-level form for structs where that trade-off is acceptable, and use field-level annotations when only specific fields are meant to be backward-compatible additions.
- `#[serde(default)]` only changes what happens on the way in. If the goal is also to omit default values on the way out, it needs to be paired with `#[serde(skip_serializing_if = "...")]` on the same field.

## References
- [serde-skip-empty](serde-skip-empty.md)
- [api-default-impl](api-default-impl.md)
