---
title: Name Types, Traits, and Enums in UpperCamelCase
impact: LOW
impactDescription: Matches rustc's non_camel_case_types lint, distinguishes types from values on sight
tags: [naming, style, types, traits]
---

# Name Types, Traits, and Enums in UpperCamelCase [LOW]

## Description
Every item that names a *type* in Rust — a struct, an enum, a trait, a type alias — uses `UpperCamelCase`, the mirror image of the `snake_case` used for functions and variables. The split exists so a reader can classify an identifier's role purely from its shape: `Connection` is obviously a type, `open_connection` is obviously a function, without needing to look up either declaration. rustc's `non_camel_case_types` lint enforces the type half of this split by default, so a struct or trait named outside the convention shows up as a compiler warning before it ever reaches review.

## Bad Example
```rust
struct http_request {     // warning: should be UpperCamelCase
    url: String,
}

trait serializable { ... } // warning

enum SCREAMING_STATE { }   // wrong on two counts
```

## Good Example
```rust
struct HttpRequest {
    url: String,
}

trait Serializable { ... }

enum ConnectionState {
    Idle,
    Active,
}

type Handler = Box<dyn Fn(&Request) -> Response>;
```

## Notes
- Type aliases follow the same rule as the types they stand for — `type Result<T> = std::result::Result<T, Error>` is `UpperCamelCase` even though it's "just" an alias.
- Acronyms inside a type name get folded in as ordinary words (`HttpServer`, not `HTTPServer`) — see `name-acronym-word` for the full reasoning and the two-letter exceptions.
- A generic type's own name stays `UpperCamelCase` even while its type parameters are single letters — `struct Cache<K, V>` combines both conventions in one declaration without conflict.

## References
- [name-acronym-word](name-acronym-word.md)
- [name-funcs-snake](name-funcs-snake.md)
- [name-variants-camel](name-variants-camel.md)
