---
title: Avoid Stringly-Typed APIs
impact: HIGH
impactDescription: Converts typo and format bugs from runtime failures into compile errors
tags: [type-safety, enums, newtypes, api-design]
---

# Avoid Stringly-Typed APIs [HIGH]

## Description
A `&str` parameter accepts any value — a typo, the wrong case, a synonym nobody standardized — and it all compiles fine, failing only when that exact code path runs in production. Enums, newtypes, and validated types push the same check to compile time or to a single construction site: a `Status` enum makes `Status::Aktivev` a compile error instead of a silent runtime mismatch, and gives the compiler an exhaustiveness check that a `match "aktive" => ...` string comparison never gets. Parse untrusted strings into the typed form once, at the boundary, and let the rest of the program work with the type.

## Bad Example
```rust
fn set_status(status: &str) {
    match status {
        "pending" => { /* ... */ }
        "active" => { /* ... */ }
        _ => panic!("unknown status"), // runtime error
    }
}

set_status("Pending"); // runtime error: wrong case
set_status("aktive");  // runtime error: typo
```

## Good Example
```rust
enum Status {
    Pending,
    Active,
    Completed,
}

fn set_status(status: Status) {
    match status {
        Status::Pending => { /* ... */ }
        Status::Active => { /* ... */ }
        Status::Completed => { /* ... */ }
    } // exhaustive -- compiler checks every case
}

set_status(Status::Pending);   // OK
// set_status(Status::Aktivev); // compile error: typo caught
```

## Notes
- Parse at the boundary with `FromStr`, once, and pass the typed value through the rest of the call graph — don't re-parse the same string repeatedly.
- Reach for a validated newtype (`Email`, `UserId`) instead of an enum when the space of valid values is too large to enumerate but still has a checkable invariant.
- `#[serde(rename_all = "snake_case")]` on an enum keeps the wire format familiar (`"user_created"`) while the Rust side stays fully typed.
- The IDE and documentation benefits compound with the type-safety ones: autocomplete on an enum lists every valid value; a `&str` parameter documents nothing.

## References
- [type-newtype-validated](type-newtype-validated.md)
- [type-enum-states](type-enum-states.md)
