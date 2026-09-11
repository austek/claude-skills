---
title: Customize Field (De)serialization with serde with Modules
impact: MEDIUM
impactDescription: Keeps the domain model's types natural instead of bending them to the wire format
tags: [serde, serialization, api-design]
---

# Customize Field (De)serialization with serde with Modules [MEDIUM]

## Description
The type that best models a value in Rust and the shape that value takes on the wire are frequently two different things — a `Duration` reads naturally as a struct with nanosecond precision, but a JSON API might expect it as a bare integer count of seconds; a byte buffer might need to travel as a base64 string. The tempting shortcut is to change the struct field to whatever type serde can already handle by default, but that leaks a wire-format concern into what should be a clean domain type. `#[serde(with = "module")]` (or its one-directional cousins `serialize_with`/`deserialize_with`) takes the opposite approach: the field keeps its ideal Rust type, and a small conversion module tells serde how to translate to and from the wire representation only at the serialization boundary.

## Bad Example
```rust
use serde::{Serialize, Deserialize};

// Forces a u64 "seconds" field instead of the natural Duration type —
// every call site now does its own manual conversion
#[derive(Serialize, Deserialize, Debug)]
struct Task {
    name: String,
    timeout_secs: u64,
}
```

## Good Example
```rust
use serde::{Serialize, Deserialize, Serializer, Deserializer};
use std::time::Duration;

mod duration_secs {
    use super::*;

    pub fn serialize<S>(duration: &Duration, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: Serializer,
    {
        serializer.serialize_u64(duration.as_secs())
    }

    pub fn deserialize<'de, D>(deserializer: D) -> Result<Duration, D::Error>
    where
        D: Deserializer<'de>,
    {
        let secs = u64::deserialize(deserializer)?;
        Ok(Duration::from_secs(secs))
    }
}

#[derive(Serialize, Deserialize, Debug)]
struct Task {
    name: String,
    // Wire format: {"name":"...","timeout":30} — Rust type stays Duration
    #[serde(with = "duration_secs", rename = "timeout")]
    timeout: Duration,
}
```

## Notes
- The module named after `with = "..."` has to provide exactly two functions with serde-defined signatures — `serialize` taking a reference to the field's type, `deserialize` returning an owned value of it — while `serialize_with`/`deserialize_with` let a single free function handle just one direction when the other can stay at its derived default.
- Because the conversion lives in one module, it's trivial to reuse across every field that shares the same representation instead of rewriting the logic per struct; several ecosystem crates (`time`, `chrono`, `uuid`) already ship a ready-made module behind their own `serde` feature flag, so it's worth checking before writing one from scratch.
- If the same custom representation shows up on fields across many structs, wrapping the value in a newtype with its own hand-written `Serialize`/`Deserialize` impl usually reads better than pasting `#[serde(with = "...")]` onto every occurrence.
- A frequent mistake when hand-writing the module is forgetting that `serialize` receives a borrowed `&T`, not an owned value — the compiler error this produces points at the trait bound rather than at the real cause.

## References
- [serde-try-from-validate](serde-try-from-validate.md)
- [type-newtype-validated](type-newtype-validated.md)
