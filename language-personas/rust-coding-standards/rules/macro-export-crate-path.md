---
title: Export Declarative Macros with macro_export and a Clean Import Path
impact: LOW
impactDescription: Lets callers import macros like any other item instead of the legacy macro_use style
tags: [macros, declarative-macros, api-design]
---

# Export Declarative Macros with macro_export and a Clean Import Path [LOW]

## Description
`macro_rules!` macros aren't visible outside their defining crate by default, and `#[macro_export]` is what changes that — it takes the macro and re-anchors it at the crate root as though it were an ordinary public item, which is what makes a normal `use path::to::macro;` import work for it at all. That matters because the alternative predates the 2018 edition's module system: `#[macro_use] extern crate mycrate;` pulled every exported macro from that crate straight into the caller's global namespace, with behavior that depended on where in the file that line happened to sit. A path-based import doesn't have either problem, and it only became possible once a macro was addressable as a normal item — which is exactly what `#[macro_export]` provides.

## Bad Example
```rust
// lib.rs — legacy style, requires callers to write
// `#[macro_use] extern crate mylib;`, polluting their global namespace
macro_rules! greet {
    ($name:expr) => {
        println!("hello, {}", $name);
    };
}
```
```rust
// consumer/src/main.rs — legacy, order-sensitive
#[macro_use]
extern crate mylib;

fn main() {
    greet!("world");
}
```

## Good Example
```rust
// lib.rs — modern style
#[macro_export]
macro_rules! greet {
    ($name:expr) => {
        println!("hello, {}", $name);
    };
}
```
```rust
// consumer/src/main.rs — modern, explicit import
use mylib::greet;

fn main() {
    greet!("world");
}
```

## Notes
- New code should default to a path import over `#[macro_use]` — it's explicit about exactly which macro is being brought in, and unlike the old style it doesn't fight with `rustfmt` or break IDE go-to-definition. The only remaining reason to reach for `#[macro_use]` is supporting a consumer still on a pre-2018 edition.
- No matter which module a `macro_rules!` definition is textually written inside, `#[macro_export]` always surfaces it at the crate root — if it also needs to be reachable through a specific module path, that requires a separate `pub use crate::greet;` re-export placed inside that module.
- Any internal helper an exported macro's expansion calls should be reached through a `$crate::__private::...` path rather than a normal public path, which keeps those helpers out of the crate's documented, semver-stable surface.

## References
- [macro-rules-hygiene](macro-rules-hygiene.md)
- [macro-private-helpers](macro-private-helpers.md)
