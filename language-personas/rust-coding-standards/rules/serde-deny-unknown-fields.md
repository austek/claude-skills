---
title: Reject Unexpected Keys with serde deny_unknown_fields
impact: HIGH
impactDescription: Turns a silently-ignored typo into a hard parse error
tags: [serde, serialization, validation]
---

# Reject Unexpected Keys with serde deny_unknown_fields [HIGH]

## Description
Serde's ordinary behavior toward keys it doesn't recognize is to simply ignore them — anything in the input that has no matching struct field is dropped without complaint. That leniency is exactly wrong for something like a hand-edited config file or a strict API contract: someone misspells a key as `"timout_secs"`, the struct deserializes without error, and the field the typo was meant to populate quietly keeps whatever default or zero value it started with. `#[serde(deny_unknown_fields)]` flips that behavior — any key the struct doesn't define now fails deserialization outright, which means the typo shows up immediately as a parse error instead of as a mysteriously-unconfigured value discovered much later at runtime.

## Bad Example
```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct ServerConfig {
    host: String,
    port: u16,
    timeout_secs: u64,
}

fn main() {
    // "timout_secs" is a typo — serde silently ignores it, timeout_secs stays 0
    let json = r#"{"host":"localhost","port":8080,"timout_secs":30}"#;
    let cfg: ServerConfig = serde_json::from_str(json).unwrap();
    println!("{cfg:?}"); // timeout_secs is 0, not 30 — no error anywhere
}
```

## Good Example
```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
#[serde(deny_unknown_fields)]
struct ServerConfig {
    host: String,
    port: u16,
    timeout_secs: u64,
}

fn parse_config(json: &str) -> Result<ServerConfig, serde_json::Error> {
    serde_json::from_str(json)
}

fn main() {
    let bad = r#"{"host":"localhost","port":8080,"timout_secs":30}"#;
    assert!(parse_config(bad).is_err()); // typo is now a hard, actionable error

    let good = r#"{"host":"localhost","port":8080,"timeout_secs":30}"#;
    assert!(parse_config(good).is_ok());
}
```

## Notes
- The best candidates for this attribute are structs where an unrecognized key almost certainly signals a mistake: configuration files, request/response DTOs, anything forming a documented API contract. It's the wrong choice for a struct deliberately designed to accept caller-supplied extra metadata — for that case, pair `#[serde(flatten)]` with a `HashMap` catch-all instead of rejecting the extra keys.
- It cannot coexist with `#[serde(flatten)]` on the same struct — `flatten` works by forwarding every key it doesn't otherwise recognize into the flattened field, and `deny_unknown_fields` claims those same unrecognized keys as errors before `flatten` ever sees them. Needing both behaviors at once usually means splitting the struct in two, or deserializing into a generic `serde_json::Value` and inspecting it by hand.
- The attribute isn't JSON-specific — it works the same way across TOML, YAML, and other self-describing formats — and the resulting error message names the specific field that didn't match, so whoever wrote the bad config gets a message they can act on directly.

## References
- [serde-flatten](serde-flatten.md)
- [api-parse-dont-validate](api-parse-dont-validate.md)
