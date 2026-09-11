---
title: Name Enum Variants in UpperCamelCase
impact: LOW
impactDescription: Matches rustc's non_camel_case_types lint, keeps variants visually distinct from fields
tags: [naming, style, enums, variants]
---

# Name Enum Variants in UpperCamelCase [LOW]

## Description
An enum variant is a type-level construct, not a value binding, so it follows the same `UpperCamelCase` rule as structs and traits rather than the `snake_case` rule for functions and locals — `Status::Pending`, not `Status::pending`. Beyond satisfying rustc's lint, the casing choice inside a variant's own name also matters: a vague variant like `Error` or a redundant one like `ConnectionError` inside `enum ConnectionState` carries less information than a specific, non-repetitive name like `TimedOut` does, even once the casing itself is correct.

## Bad Example
```rust
enum ConnectionState {
    connected,           // warning: should be UpperCamelCase
    CONNECTION_ERROR,     // wrong casing, and redundant with the enum's own name
}
```

## Good Example
```rust
enum ConnectionState {
    Connected,
    Disconnected,
    TimedOut,
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },       // struct-style variant, still UpperCamelCase
    Write(String),                  // tuple-style variant, same rule
}
```

## Notes
- Struct-style and tuple-style variants follow the identical casing rule as unit variants — the variant's *payload shape* doesn't change how its name is cased.
- Repeating the enum's own name inside a variant (`ConnectionState::ConnectionError`) is usually a sign the variant name should be shortened, since the enum name already supplies that context at every use site (`ConnectionState::TimedOut` reads just as clearly).
- An `Option`-shaped custom enum should still spell out `Some(T)`/`None`-style variants distinctly rather than reusing `Some`/`None` themselves, which would shadow the real `Option` variants inside the same scope.

## References
- [name-types-camel](name-types-camel.md)
