---
title: Enforce cargo fmt --check in CI
impact: MEDIUM
impactDescription: Removes formatting disagreements from PR review entirely
tags: [tooling, rustfmt, ci, formatting]
---

# Enforce cargo fmt --check in CI [MEDIUM]

## Description
`cargo fmt --check` exits non-zero if any file in the crate would be reformatted by `cargo fmt`, without actually touching any file — which makes it exactly the assertion a CI pipeline needs. Wiring it in means a PR that hasn't been formatted fails a fast, cheap check before a human reviewer ever looks at the diff, instead of a reviewer spending review time on whitespace and line-break nitpicks that a formatter should have settled automatically. It converts "please run rustfmt" from a recurring comment on every third PR into a rule the tooling enforces.

## Bad Example
```yaml
# CI pipeline with no formatting check — style drift accumulates,
# and inconsistently formatted diffs make every PR harder to review.
jobs:
  test:
    steps:
      - run: cargo test
```

## Good Example
```yaml
# .github/workflows/ci.yml
jobs:
  fmt:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
        with:
          components: rustfmt
      - run: cargo fmt --all --check
```

## Notes
- `--all` matters in a workspace — without it, `cargo fmt --check` only checks the crate in the current directory, silently skipping other workspace members.
- A checked-in `rustfmt.toml` (with settings like `max_width`, `imports_granularity`, `group_imports`) makes the formatting deterministic across every contributor's machine and CI, so `--check` failures are always about real deviations, never about a contributor's local rustfmt defaults differing from the project's.
- `#[rustfmt::skip]` on a specific item (a hand-aligned matrix literal, generated code) is the correct escape hatch for code that genuinely reads better un-formatted — reach for it instead of letting the whole file drift out of sync with `cargo fmt --all`.
- Some `rustfmt.toml` options (`imports_granularity = "Crate"`, `wrap_comments`) are nightly-only; a CI job running stable `rustfmt` will silently ignore them, so verify which toolchain your CI check actually runs against if you rely on those settings.

## References
- [lint-warn-style](lint-warn-style.md)
- [lint-pedantic-selective](lint-pedantic-selective.md)
