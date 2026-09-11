---
title: Skip the -rs / -rust Suffix on Crate Names
impact: LOW
impactDescription: Frees a cleaner name on crates.io and avoids redundant characters
tags: [naming, crates, ecosystem, style]
---

# Skip the -rs / -rust Suffix on Crate Names [LOW]

## Description
A crate published on crates.io is unambiguously a Rust crate — tacking `-rs` or `-rust` onto its name states something already implied by where it lives, at the cost of extra characters in every `Cargo.toml` dependency line and every `use` path a downstream crate writes. It also tends to claim the plain, more desirable name for nobody, since a would-be `json-parser` crate now competes visually with the taken `json-parser-rs`. Widely used crates avoid the suffix entirely: `serde`, `tokio`, `reqwest`, `clap` — none of them append a language tag to their own name.

## Bad Example
```toml
[package]
name = "yaml-config-rs"   # redundant: crates.io is already Rust-only
```

## Good Example
```toml
[package]
name = "yaml-config"
```

## Notes
- The suffix earns its place when disambiguating from a well-known non-Rust project of the same name, or for official crates under the `rust-lang` organization that intentionally mirror a non-Rust original.
- A binding or port of an existing library is better named after what it wraps or does (`openssl`, `python-ast`) than after the fact that it's written in Rust.
- The same logic extends to repository names on GitHub — a trailing `-rs` there is rarely load-bearing information for a visitor who already knows they're on a Rust project page.

## References
- [name-funcs-snake](name-funcs-snake.md)
