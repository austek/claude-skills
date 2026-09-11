---
title: Configure Lints Once at the Workspace Level
impact: MEDIUM
impactDescription: Eliminates per-crate lint drift across every member of a workspace
tags: [tooling, workspace, lints, ci, consistency]
---

# Configure Lints Once at the Workspace Level [MEDIUM]

## Description
In a multi-crate workspace with no central lint policy, every crate ends up with whatever configuration its author happened to add — one crate denies `unwrap_used`, another has no lint configuration at all, and enforcement quietly depends on which contributor set up which crate. Since Rust 1.74, `[workspace.lints.*]` tables in the root `Cargo.toml` let you declare lint levels once and have every member crate inherit them with a single `[lints] workspace = true`, so the whole workspace shares one enforced standard instead of an accidental patchwork. A crate that genuinely needs a different level for a specific lint (a binary crate allowing `unwrap_used` at its entry point, say) can still override that one lint locally without abandoning the shared baseline for everything else.

## Bad Example
```toml
# crate-a/Cargo.toml — strict
[lints.clippy]
unwrap_used = "deny"

# crate-b/Cargo.toml — nothing configured at all
# crate-c/Cargo.toml — different level for the same lint
[lints.clippy]
unwrap_used = "warn"
```

## Good Example
```toml
# Root Cargo.toml
[workspace.lints.rust]
unsafe_code = "deny"

[workspace.lints.clippy]
unwrap_used = "deny"
expect_used = "warn"

# crate-a/Cargo.toml
[lints]
workspace = true

# crate-b/Cargo.toml
[lints]
workspace = true
```

## Notes
- `{ level = "deny", priority = -1 }` lets you set a whole group (`correctness`, `suspicious`) at once while still overriding individual lints within it at a normal (higher) priority — the group applies first, then the specific overrides win.
- A binary crate that legitimately wants a looser rule than the workspace default (allowing `unwrap_used` at a `main.rs` entry point where a panic is an acceptable failure mode) can still set `[lints] workspace = true` and then add its own `[lints.clippy] unwrap_used = "allow"` on top — the per-crate override takes precedence over the inherited workspace setting.
- Run `cargo clippy --workspace --all-targets -- -D warnings` in CI, not just `cargo clippy` in each crate directory — the `--workspace` flag is what actually applies the shared configuration across every member rather than just whichever crate happens to be the current directory.
- Centralizing lints doesn't remove the need to choose sensible defaults — a workspace-wide `deny` on a lint the team hasn't actually agreed on just moves the friction from "unenforced inconsistency" to "one crate's CI blocked on a rule nobody signed off on."

## References
- [lint-deny-correctness](lint-deny-correctness.md)
- [proj-workspace-deps](proj-workspace-deps.md)
- [anti-unwrap-abuse](../../rust-coding-standards/rules/anti-unwrap-abuse.md)
