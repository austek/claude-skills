---
title: Keep build.rs Minimal, Deterministic, and Idempotent
impact: HIGH
impactDescription: Preserves incremental builds, offline builds, and reproducibility that a careless build script silently breaks
tags: [tooling, build-script, cargo, reproducibility]
---

# Keep build.rs Minimal, Deterministic, and Idempotent [HIGH]

## Description
A `build.rs` script runs before every Cargo command that touches its crate, and Cargo decides whether to skip re-running it based entirely on the `cargo::rerun-if-changed` and `cargo::rerun-if-env-changed` directives the script itself emits. Emit none, and Cargo falls back to re-running the script on literally every build; emit directives that are too broad, and you force needless recompiles that have nothing to do with what actually changed. Beyond that re-run logic, anything non-deterministic inside the script — embedding a timestamp, generating a random UUID, reaching out over the network — breaks reproducible builds and offline development, since two builds of the identical source can now produce different output, or fail outright with no network access. Detecting compiler capabilities by parsing the output of `rustc --version` is a related trap: it's fragile against version-string format changes, where the `autocfg` crate's approach — actually compiling a tiny probe snippet against the real toolchain — tells you the truth regardless of how the version string happens to be formatted.

## Bad Example
```rust
// build.rs — no rerun directives (re-runs on every build), and a network
// call that breaks offline and reproducible builds.
fn main() {
    let output = std::process::Command::new("rustc").arg("--version").output().unwrap();
    let version = String::from_utf8(output.stdout).unwrap();
    if version.contains("1.8") {
        println!("cargo::rustc-cfg=has_feature"); // fragile: parses a string, not a real probe
    }
    let _resp = reqwest::blocking::get("https://example.com/schema.json").unwrap();
}
```

## Good Example
```rust
// build.rs — narrow rerun scope, real capability probe, no network access.
fn main() {
    println!("cargo::rerun-if-changed=build.rs");
    println!("cargo::rerun-if-changed=src/generated.rs");

    let ac = autocfg::new();
    ac.emit_has_type("std::collections::BTreeMap");
}
```

```toml
[build-dependencies]
autocfg = "1"
```

## Notes
- At minimum, always emit `cargo::rerun-if-changed=build.rs` — without it, Cargo's fallback behavior is to re-run the script on every single build regardless of what changed, which is the exact non-selectivity you're trying to avoid.
- List every input the script actually reads — schema files, codegen templates, anything beyond the script itself — since a missing entry means Cargo won't notice a relevant change and you'll ship stale generated output.
- Treat a network call inside `build.rs` as a hard no: vendor whatever the script needs into the repository (or behind an explicit opt-in feature) instead, so the build works offline and produces the same output every time it runs.
- Anything the script writes must stay inside `OUT_DIR` (the path Cargo hands the script for exactly this purpose) — writing elsewhere breaks build environments that sandbox filesystem access outside that directory.

## References
- [proj-feature-additive](proj-feature-additive.md)
- [lint-cfg-check](lint-cfg-check.md)
