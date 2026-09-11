---
title: Deny clippy::correctness Instead of Warning
impact: CRITICAL
impactDescription: Blocks merges of code clippy has proven is outright wrong, not just stylistically off
tags: [tooling, clippy, correctness, ci]
---

# Deny clippy::correctness Instead of Warning [CRITICAL]

## Description
`clippy::correctness` is the one clippy group that doesn't deal in style or preference — every lint in it flags code that is actually wrong: comparing a float to `NaN` with `==` (always false), an infinite iterator fed straight into a `for` loop, a reference that outlives the value it points to inside the same block. Leaving these at `warn` means they show up in build output but don't stop a merge, which in practice means they get scrolled past exactly as often as any other warning noise. `deny` turns them into hard compile failures, which is the correct treatment for a class of lint that clippy is not guessing about — it has already determined the code cannot behave as the author intended.

## Bad Example
```rust
// Compiles and only warns at the default level — easy to merge by accident.
if x == f64::NAN {  // NaN != NaN always; this branch can never run
    // unreachable in practice
}
```

## Good Example
```toml
# Cargo.toml — correctness violations fail the build, not just the linter output.
[lints.clippy]
correctness = "deny"
```

```rust
#![deny(clippy::correctness)]

// Now the NaN comparison above is a hard compiler error, not a warning
// that's easy to miss in a long CI log.
```

## Notes
- Set this at the crate root (`#![deny(clippy::correctness)]` in `lib.rs`/`main.rs`) or centrally via `[lints.clippy] correctness = "deny"` in `Cargo.toml` — the `Cargo.toml` form additionally applies without needing the attribute repeated in every crate root of a workspace.
- Pair `deny(correctness)` with `warn` on the `suspicious`, `style`, `complexity`, and `perf` groups rather than denying those too — correctness lints are proven bugs, while the other groups include some lints reasonable teams choose to disable, so denying them removes that flexibility.
- `cargo clippy -- -D warnings` in CI is a coarser hammer that denies everything clippy would otherwise only warn about; `deny(clippy::correctness)` embedded in the source is the more durable choice since it travels with the code rather than depending on the exact CI invocation.
- A correctness lint firing on code that genuinely needs the flagged pattern (rare, but it happens with things like `#[allow(clippy::approx_constant)]` for a domain-specific constant) should be suppressed at the smallest possible scope with a comment explaining why, not by dropping the group to `warn` project-wide.

## References
- [lint-warn-suspicious](lint-warn-suspicious.md)
- [lint-warn-perf](lint-warn-perf.md)
