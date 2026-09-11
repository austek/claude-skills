---
title: Put Proc-Macros in a Dedicated Crate, Re-Export from a Facade
impact: MEDIUM
impactDescription: Lets users depend on one crate while the proc-macro implementation stays hidden
tags: [macros, proc-macros, crate-structure]
---

# Put Proc-Macros in a Dedicated Crate, Re-Export from a Facade [MEDIUM]

## Description
Setting `proc-macro = true` in a crate's `Cargo.toml` changes what that crate is allowed to compile to: it now builds for the host machine running the compiler rather than the eventual target, and Cargo restricts it to exporting proc-macros only — no plain functions, traits, or structs alongside them. A library that wants to offer both a derive macro and a normal API therefore can't be a single crate; the usual answer is two crates, a `proc-macro = true` crate holding just the macro implementation, and a second "facade" crate that depends on the first and re-exports its macro next to the library's ordinary public API. Users of the library only ever add the facade crate as a dependency, code the macro generates refers back to items through `::mycrate::__private::...` so the implementation crate's own version number never leaks into generated code, and putting both crates in one Cargo workspace with a shared `[workspace.dependencies]` entry keeps their versions moving together automatically.

## Bad Example
```rust
// A single crate with proc-macro = true that also tries to export regular items
#[proc_macro_derive(Greet)]
pub fn derive_greet(input: TokenStream) -> TokenStream { /* ... */ }

pub trait Greet { fn greet(&self) -> String; } // error: a proc-macro crate
pub struct Config;                              // can only export proc-macros
```

## Good Example
```toml
# mycrate-derive/Cargo.toml
[package]
name = "mycrate-derive"
edition = "2024"

[lib]
proc-macro = true   # required — makes this a proc-macro-only crate

[dependencies]
syn.workspace = true
quote.workspace = true
```
```toml
# mycrate/Cargo.toml
[package]
name = "mycrate"
edition = "2024"

[dependencies]
mycrate-derive.workspace = true
```
```rust
// mycrate/src/lib.rs — the facade re-exports the derive and defines the real API
pub use mycrate_derive::Greet;

pub trait Greet {
    fn greet(&self) -> String;
}

#[doc(hidden)]
pub mod __private {
    pub fn format_greeting(name: &str) -> String {
        format!("hello, {name}")
    }
}
```
```rust
// Users depend only on `mycrate`:
use mycrate::Greet;
#[derive(mycrate::Greet)]
struct Robot;
```

## Notes
- Code the derive crate generates always addresses the facade crate through an absolute path like `::mycrate::__private::...`, never a relative one — the macro's own compilation context and the context the generated code actually runs in are two different crates, and only an absolute path is guaranteed to resolve correctly in both.
- Declaring `mycrate-derive` under `[workspace.dependencies]` and referencing it from the facade as `mycrate-derive.workspace = true` avoids having to bump a version number in two `Cargo.toml` files every time the pair is released together.
- Most users of `thiserror` never realize it's actually two crates under the hood — a `proc-macro = true` implementation crate plus a thin facade — which is a good demonstration of how invisible this split can be from the outside when it's done right.

## References
- [macro-proc-syn-quote](macro-proc-syn-quote.md)
- [macro-private-helpers](macro-private-helpers.md)
- [err-thiserror-lib](err-thiserror-lib.md)
