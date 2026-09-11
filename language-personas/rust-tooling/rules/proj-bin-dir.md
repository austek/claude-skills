---
title: Put Multiple Binaries in src/bin/
impact: LOW
impactDescription: Removes manual [[bin]] bookkeeping entirely for the common case
tags: [tooling, project-structure, binaries, cargo]
---

# Put Multiple Binaries in src/bin/ [LOW]

## Description
A crate that ships more than one executable has two ways to tell Cargo about them: hand-write a `[[bin]]` table per binary in `Cargo.toml`, or drop each entry point as its own file under `src/bin/` and let Cargo discover them automatically. The convention-based approach wins by default — every `.rs` file directly in `src/bin/` becomes a binary target named after the file, with zero `Cargo.toml` changes required to add or remove one. It also removes an entire category of confusion in `src/`: a flat `src/` with `main.rs`, `server.rs`, and `cli.rs` sitting side by side gives no visual signal about which files are binaries and which are library modules, whereas `src/bin/server.rs` is unambiguous on sight.

## Bad Example
```toml
# Cargo.toml — manually wiring up two binaries that src/bin/ would find for free.
[[bin]]
name = "server"
path = "src/server.rs"

[[bin]]
name = "cli"
path = "src/cli.rs"
```

## Good Example
```
src/
├── lib.rs        # shared library code
└── bin/
    ├── server.rs # binary: server
    └── cli.rs    # binary: cli
```

```rust
// src/lib.rs — shared code
pub mod config;
pub mod database;

// src/bin/server.rs
use my_project::{config, database};

fn main() {
    let cfg = config::load();
    let _db = database::connect(&cfg);
}
```

## Notes
- Each binary in `src/bin/` should stay a thin entry point that calls into `lib.rs` — the same reasoning that keeps `main.rs` minimal applies per-binary here, since code inside `src/bin/*.rs` is unreachable from integration tests.
- `cargo run --bin server`, `cargo build --bin cli`, and `cargo build --bins` (all of them at once) are the standard commands for working with a multi-binary crate; plain `cargo run` only works unambiguously when there's a single binary or a `default-run` is set.
- A binary complex enough to span multiple files becomes a directory instead of a file: `src/bin/server/main.rs` plus sibling modules in that same directory still compiles to a binary named `server`.
- Fall back to explicit `[[bin]]` entries only when a binary needs something the convention can't express — a custom name that doesn't match its filename, or `required-features` gating it behind a Cargo feature.

## References
- [proj-lib-main-split](proj-lib-main-split.md)
- [proj-workspace-large](proj-workspace-large.md)
- [proj-flat-small](proj-flat-small.md)
