---
title: Implement FromStr Instead of a Bespoke Parse Function
impact: MEDIUM
impactDescription: Unlocks .parse::<T>() and integrates with clap, serde, and generic bounds
tags: [conversions, from-str, parsing, api-design]
---

# Implement FromStr Instead of a Bespoke Parse Function [MEDIUM]

## Description
`FromStr` is the single standard hook for turning a `&str` into a typed value. Implementing it unlocks the idiomatic `.parse::<T>()` call, is picked up automatically by CLI argument parsers like `clap`, and is the interface generic code expects when it bounds a parameter with `T: FromStr`. A hand-rolled `fn parse_foo(s: &str)` forces every caller to learn a private, crate-specific name and is invisible to any of that ecosystem tooling — the conversion works, but only through a door nothing else knows to look for.

## Bad Example
```rust
#[derive(Debug)]
enum Color { Red, Green, Blue }

// Callers must know this exact name; no .parse() support, no clap integration.
fn parse_color(s: &str) -> Result<Color, String> {
    match s {
        "red" => Ok(Color::Red),
        "green" => Ok(Color::Green),
        "blue" => Ok(Color::Blue),
        other => Err(format!("unknown color: {other}")),
    }
}
```

## Good Example
```rust
use std::str::FromStr;

#[derive(Debug, PartialEq)]
enum Color { Red, Green, Blue }

#[derive(Debug)]
struct ParseColorError(String);

impl FromStr for Color {
    type Err = ParseColorError;

    fn from_str(s: &str) -> Result<Self, Self::Err> {
        match s {
            "red" => Ok(Color::Red),
            "green" => Ok(Color::Green),
            "blue" => Ok(Color::Blue),
            other => Err(ParseColorError(other.to_owned())),
        }
    }
}

// Standard idiom -- works with clap, config parsers, generic T: FromStr code.
let c: Color = "green".parse().unwrap();
```

## Notes
- Use a concrete `Err` type, not `String`, so callers can pattern-match on the specific failure rather than string-matching a message.
- `FromStr` pairs naturally with `Display` — if a type can be parsed in, it should generally be printable back out in the same format.
- For infallible string conversions (wrapping a `String` in a newtype with no validation), `From<&str>`/`From<String>` is the better fit than a `FromStr` that never actually errors.
- `clap`'s `value_parser` attribute macro detects `FromStr` automatically — implementing it once gives CLI argument parsing for free.

## References
- [conv-tryfrom-fallible](conv-tryfrom-fallible.md)
- [type-newtype-validated](type-newtype-validated.md)
- [api-parse-dont-validate](api-parse-dont-validate.md)
