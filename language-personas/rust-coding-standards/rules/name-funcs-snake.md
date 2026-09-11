---
title: Name Functions, Methods, and Variables in snake_case
impact: LOW
impactDescription: Matches rustc's non_snake_case lint, keeps value-level names visually distinct from types
tags: [naming, style, functions, variables]
---

# Name Functions, Methods, and Variables in snake_case [LOW]

## Description
Rust splits its naming conventions by what kind of item is being named: types get `UpperCamelCase`, and everything that exists at the value level — functions, methods, local bindings, module names — gets `snake_case`. That split is not arbitrary; it means a reader can tell from casing alone, without any other context, whether `Connection` names a type or `open_connection` names a function. rustc enforces the value-level half of this with the `non_snake_case` lint, which warns by default whenever a function or variable uses camelCase or PascalCase.

## Bad Example
```rust
fn fetchUserProfile(userId: u64) -> UserProfile { // warning: should be snake_case
    let LastLogin = userId; // also warns, and wrong semantics anyway
    todo!()
}
```

## Good Example
```rust
fn fetch_user_profile(user_id: u64) -> UserProfile {
    let last_login = user_id;
    todo!()
}

mod user_service;
mod http_client;
```

## Notes
- An acronym inside a `snake_case` name is lowercased along with everything else: `parse_json`, `connect_tcp`, never `parse_JSON`.
- Module names follow the same rule as functions — `mod http_client`, not `mod HttpClient` — because a module is a value-level namespace, not a type.
- This convention is one of the few in Rust naming where mixing styles within one function signature (e.g. a camelCase parameter next to a snake_case local) reliably signals code that was ported from another language without adaptation.

## References
- [name-types-camel](name-types-camel.md)
- [name-consts-screaming](name-consts-screaming.md)
