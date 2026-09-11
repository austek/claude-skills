---
title: Name Generic Type Parameters with a Single Uppercase Letter
impact: LOW
impactDescription: Keeps generic signatures short and instantly recognizable
tags: [naming, style, generics, type-parameters]
---

# Name Generic Type Parameters with a Single Uppercase Letter [LOW]

## Description
Rust's generics convention uses one uppercase letter per type parameter, with a small fixed vocabulary of which letter means what: `T` for a generic type, `E` for an error, `K`/`V` for a map's key and value, `F` for a function or closure. These aren't arbitrary — they mirror math notation's tradition of single-letter variables, and because the vocabulary is small and consistent, a reader recognizes `Result<T, E>` or `HashMap<K, V>` as a pattern rather than parsing it fresh every time. A full descriptive name like `ElementType` isn't wrong exactly, but it reads as unfamiliar where the community has already agreed on `T`.

## Bad Example
```rust
struct Cache<KeyType, ValueType> {
    entries: HashMap<KeyType, ValueType>,
}

fn transform<InputType, OutputType>(x: InputType) -> OutputType { todo!() }
```

## Good Example
```rust
struct Cache<K, V> {
    entries: HashMap<K, V>,
}

fn transform<I, O>(x: I) -> O { todo!() }
```

## Notes
- When a signature genuinely needs more than three or four type parameters, that's usually a sign the type or function is doing too much — reach for `codebase-design`-style decomposition before reaching for longer parameter names as a workaround.
- Keep bounds in a `where` clause rather than inline on the parameter list (`fn f<T>(x: T) where T: Clone + Debug`), so the short parameter names don't get buried under a wall of trait bounds on the same line.
- `A` for an allocator type parameter (as in `Vec<T, A>`) is a newer addition to the vocabulary, following the same single-letter pattern as the allocator API stabilizes.

## References
- [name-lifetime-short](name-lifetime-short.md)
- [name-types-camel](name-types-camel.md)
