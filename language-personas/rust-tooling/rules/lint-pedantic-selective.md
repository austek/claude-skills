---
title: Cherry-Pick clippy::pedantic Lints Instead of the Whole Group
impact: LOW
impactDescription: Keeps the useful pedantic checks while dropping the ones teams disagree on
tags: [tooling, clippy, pedantic, lints]
---

# Cherry-Pick clippy::pedantic Lints Instead of the Whole Group [LOW]

## Description
`clippy::pedantic` is a large group of opinionated, defensible-but-debatable lints — things like flagging functions over an arbitrary line count, or requiring an `# Errors` doc section on every fallible function. None of them are wrong exactly, but few teams agree with every single one, and enabling the whole group as-is tends to produce a wall of warnings that gets `#[allow]`'d away wholesale rather than engaged with individually. Treating pedantic as a menu — turn the group on, then deliberately `allow` the subset that doesn't fit the project, or skip the group entirely and hand-pick only the lints you want — keeps the signal-to-noise ratio high enough that the surviving warnings actually get acted on.

## Bad Example
```rust
// Enables the entire group at once — expect constant, largely unreviewed
// friction until someone eventually silences most of it in frustration.
#![warn(clippy::pedantic)]
```

## Good Example
```toml
# Cargo.toml
[lints.clippy]
pedantic = "warn"

# Explicitly opt out of the ones that don't fit this project.
missing_errors_doc = "allow"      # tracked separately via a doc policy
module_name_repetitions = "allow" # Foo::FooError is intentional here
too_many_lines = "allow"          # function length varies by domain
```

## Notes
- A workable starter subset tends to include `doc_markdown` (catches unmarked code identifiers inside doc comments), `unused_self` (a method that never touches `self` probably should be a free function), and `wildcard_imports` (glob imports hide where a name actually comes from) — these rarely produce false positives and catch real readability issues.
- `missing_errors_doc` and `missing_panics_doc` are commonly disabled not because documenting error and panic conditions is unimportant, but because many teams handle that documentation through a different convention (a dedicated `# Errors` policy enforced in review) rather than clippy.
- Decide as a team before enabling the group wholesale — pedantic lints encode style opinions, and a lint silently `#[allow]`'d by one contributor after a single annoying encounter effectively becomes dead configuration nobody remembers exists.
- The alternative to enabling the group and then carving out exceptions is enabling only the specific lints you want from the start — better when your subset is small, since it avoids inheriting new pedantic lints automatically on every clippy upgrade.

## References
- [lint-warn-style](lint-warn-style.md)
- [lint-warn-complexity](lint-warn-complexity.md)
- [lint-deny-correctness](lint-deny-correctness.md)
