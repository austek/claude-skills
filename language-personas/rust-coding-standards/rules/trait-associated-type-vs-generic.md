---
title: Choose Associated Types for One Output, Generics for Many
impact: MEDIUM
impactDescription: Avoids either turbofish noise or an impossible second impl
tags: [trait-design, associated-types, generics, api-design]
---

# Choose Associated Types for One Output, Generics for Many [MEDIUM]

## Description
An associated type (`type Output;`) is part of the implementing type's identity — there can be exactly one binding per impl, so callers never need turbofish to disambiguate it, matching how `Iterator::Item` and `Future::Output` work. A generic parameter (`<Rhs>`) allows multiple simultaneous impls of the same trait on one type, which is exactly what `Add<Vec2>` and `Add<f64>` both implemented for `Vec2` require — something an associated type can never express. Picking the wrong one either bans a legitimate second impl or forces every call site to annotate a type that was never actually ambiguous.

## Bad Example
```rust
// A generic parameter for a trait that only ever has one output per type
// forces every caller to write out the redundant type argument.
trait Parser<Output> {
    fn parse(&self, input: &str) -> Option<Output>;
}

struct JsonParser;
impl Parser<String> for JsonParser {
    fn parse(&self, input: &str) -> Option<String> {
        Some(input.to_owned())
    }
}

fn run<P: Parser<String>>(p: &P, s: &str) -> Option<String> {
    p.parse(s)
}
```

## Good Example
```rust
// Associated type: exactly one Output per implementor, no turbofish needed.
trait Parser {
    type Output;
    fn parse(&self, input: &str) -> Option<Self::Output>;
}

struct NumberParser;
impl Parser for NumberParser {
    type Output = f64;
    fn parse(&self, input: &str) -> Option<f64> {
        input.trim().parse().ok()
    }
}

fn run<P: Parser>(p: &P, s: &str) -> Option<P::Output> {
    p.parse(s)
}
```

## Notes
- `Iterator`, `Future`, and `Deref` all use associated types because there is exactly one `Item`/`Output`/`Target` per implementor.
- `Add<Rhs>`, `From<T>`, and `Into<T>` use generic parameters precisely because one type can add to, or convert from, many others.
- Constrain an associated type in a bound with `P: Parser<Output = JsonValue>` — cleaner than reaching for a free generic parameter to express the same constraint.
- If a second impl for the same trait on the same type is ever plausible, that's the signal to use a generic parameter instead of an associated type.

## References
- [type-generic-bounds](type-generic-bounds.md)
- [trait-default-methods](trait-default-methods.md)
- [api-impl-fromiterator](api-impl-fromiterator.md)
