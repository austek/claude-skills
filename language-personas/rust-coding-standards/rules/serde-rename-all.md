---
title: Match External Naming with serde rename_all
impact: LOW
impactDescription: One container attribute replaces a per-field rename on every field
tags: [serde, serialization, naming, api-design]
---

# Match External Naming with serde rename_all [LOW]

## Description
Idiomatic Rust field names are `snake_case`, but plenty of the formats Rust code has to interoperate with weren't designed with that convention in mind — JSON APIs and GraphQL schemas commonly use `camelCase`, and some config formats prefer `kebab-case` or `SCREAMING_SNAKE_CASE`. It's possible to bridge the gap field by field with `#[serde(rename = "...")]`, but that means writing one attribute per field and remembering to add another every time the struct grows — a maintenance burden that scales with the struct instead of staying constant. `#[serde(rename_all = "camelCase")]` on the container solves the whole struct in one line: every field's name is transformed by the same rule during both serialization and deserialization, and the Rust source keeps reading as normal Rust the whole time.

## Bad Example
```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize)]
struct UserProfile {
    #[serde(rename = "firstName")]
    first_name: String,
    #[serde(rename = "lastName")]
    last_name: String,
    #[serde(rename = "emailAddress")]
    email_address: String,
    #[serde(rename = "isActive")]
    is_active: bool,
}
```

## Good Example
```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize)]
#[serde(rename_all = "camelCase")]
struct UserProfile {
    first_name: String,
    last_name: String,
    email_address: String,
    is_active: bool,
    // Per-field override for a reserved word — rename_all can't rename to "type"
    #[serde(rename = "type")]
    user_type: String,
}

#[derive(Serialize, Deserialize)]
#[serde(rename_all = "SCREAMING_SNAKE_CASE")]
enum Status {
    Active,
    Inactive,
    PendingVerification,
}
```

## Notes
- The attribute isn't limited to structs — it renames enum variants the same way it renames struct fields, and it covers both directions of conversion, not just serialization. Beyond `camelCase`, serde recognizes `PascalCase`, `kebab-case`, `SCREAMING_SNAKE_CASE`, `UPPERCASE`, and `lowercase`.
- When one specific field needs a name the blanket rule wouldn't produce — a Rust keyword like `type` being the classic case — a field-level `#[serde(rename = "...")]` takes precedence over whatever `rename_all` would otherwise compute, so the two attributes combine naturally rather than conflicting.
- Don't guess the convention from what "feels standard" — check the actual payloads or schema documentation for the API being targeted, since some services aren't internally consistent and mix naming conventions across different endpoints.

## References
- [serde-default-compat](serde-default-compat.md)
- [api-serde-optional](api-serde-optional.md)
