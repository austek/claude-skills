---
title: Choose Serde Enum Tagging Deliberately
impact: MEDIUM
impactDescription: Matches the wire format to the consumer's schema instead of guessing serde's default
tags: [serde, serialization, enums, api-design]
---

# Choose Serde Enum Tagging Deliberately [MEDIUM]

## Description
When serde derives `Serialize`/`Deserialize` for an enum with no extra attributes, it wraps whichever variant is active in an object keyed by that variant's name — so a `Circle` variant becomes `{"Circle": {...}}`. That's a perfectly fine shape for data that only ever round-trips between two Rust programs, but it rarely matches what an external REST API, event schema, or config format actually expects, which is usually some kind of discriminator field alongside the data rather than a wrapper object named after the variant. Serde supports three other tagging strategies precisely because "the wire schema I need to match" and "the default serde gives me" are frequently different things, and picking the wrong one doesn't just look ugly — it produces a struct that either fails to parse the consumer's payloads or drops information on a round trip.

## Bad Example
```rust
use serde::{Serialize, Deserialize};

// Default (externally tagged): {"Circle":{"radius":5.0}}
// Most REST APIs expect {"type":"circle","radius":5.0} instead — this mismatches on arrival.
#[derive(Serialize, Deserialize, Debug)]
enum Shape {
    Circle { radius: f64 },
    Rectangle { width: f64, height: f64 },
}
```

## Good Example
```rust
use serde::{Serialize, Deserialize};

// Internally tagged — {"type":"Circle","radius":5.0}
// Matches REST APIs with a discriminator field; every variant must be a struct/map.
#[derive(Serialize, Deserialize, Debug)]
#[serde(tag = "type")]
enum ShapeInternal {
    Circle { radius: f64 },
    Rectangle { width: f64, height: f64 },
}

// Adjacently tagged — {"t":"Circle","c":{"radius":5.0}}
// Needed when a variant holds a primitive or tuple, which internal tagging can't represent.
#[derive(Serialize, Deserialize, Debug)]
#[serde(tag = "t", content = "c")]
enum ShapeAdjacent {
    Circle { radius: f64 },
    Count(u32),
}

// Untagged — {"radius":5.0}
// For a small set of structurally distinct types; tried in declaration order.
#[derive(Serialize, Deserialize, Debug)]
#[serde(untagged)]
enum Value {
    Integer(i64),
    Float(f64),
    Text(String),
}
```

## Notes
- The four options aren't interchangeable in what they can represent: the default (externally tagged) handles tuple variants fine; internally tagged (`tag = "type"`) cannot, because every variant has to serialize as a map so the tag field can sit alongside its contents; adjacently tagged (`tag`/`content`) gets tuple-variant support back by wrapping the payload in its own separate envelope field; untagged drops any explicit tag and just tries each variant's shape in turn.
- Untagged is the one to be most cautious with — since there's no discriminator to guide deserialization, serde tries every variant in declaration order until one parses successfully, which is slower than the tagged forms, can silently choose the wrong variant when two shapes overlap, and gives a vague error when nothing matches. It's a reasonable fit only for a small set of variants whose shapes are obviously distinct from each other, like a number versus a string.
- If a variant needs to carry a bare primitive or a `Vec` rather than a struct-like payload, internally tagged simply can't express that — adjacently tagged is the fallback that keeps a discriminator while still allowing non-map variant contents.

## References
- [type-enum-states](type-enum-states.md)
- [api-non-exhaustive](api-non-exhaustive.md)
- [serde-flatten](serde-flatten.md)
