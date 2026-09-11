---
title: Declare Custom Cfgs and Enable unexpected_cfgs
impact: HIGH
impactDescription: Turns a silently-dead feature gate into a compile-time warning
tags: [tooling, lints, cfg, feature-flags, compiler]
---

# Declare Custom Cfgs and Enable unexpected_cfgs [HIGH]

## Description
`#[cfg(...)]` conditions are evaluated by name matching against a string the compiler otherwise has no way to spell-check — misspell a feature (`"serde_"` instead of `"serde"`) or reference a custom cfg flag that nothing ever declared (`tokio_unstable`), and the block silently vanishes from the build with zero diagnostic output. The `unexpected_cfgs` lint closes that gap by requiring every cfg name and value to be known ahead of time: Cargo registers your `[features]` automatically, but any other custom cfg has to be listed explicitly via `check-cfg` in `[lints.rust]`, after which an unrecognized cfg produces a compiler warning instead of quietly compiling to nothing. The underlying `--check-cfg` compiler flag was stabilized in Rust 1.79.

## Bad Example
```rust
// "serde_" doesn't match the real "serde" feature — this impl block is
// dead code, and nothing about the build tells you that.
#[cfg(feature = "serde_")]
impl serde::Serialize for MyType {}
```

## Good Example
```toml
# Cargo.toml
[lints.rust]
unexpected_cfgs = { level = "warn", check-cfg = [
    'cfg(tokio_unstable)',
] }
```

```rust
#[cfg(feature = "serde")] // correct — and Cargo already knows this feature exists
impl serde::Serialize for MyType {}

#[cfg(tokio_unstable)] // declared above, so no unexpected_cfgs warning
pub fn experimental() {}
```

## Notes
- Feature names never need to be listed manually in `check-cfg` — Cargo derives the known set directly from `[features]` in `Cargo.toml` and feeds it to the compiler automatically.
- Each `check-cfg` entry is a quoted cfg expression: `'cfg(name)'` for a bare flag, or `'cfg(name, values("a", "b"))'` when the cfg also takes specific values.
- In a workspace, set `unexpected_cfgs` once under `[workspace.lints.rust]` and have each member opt in with `[lints] workspace = true`, rather than repeating the `check-cfg` list in every crate.
- The lint defaults to `warn`; promote it to `deny` in CI once the codebase is clean, so a future typo fails the build instead of merging silently.

## References
- [lint-workspace-lints](lint-workspace-lints.md)
- [proj-feature-additive](proj-feature-additive.md)
- [lint-warn-suspicious](lint-warn-suspicious.md)
