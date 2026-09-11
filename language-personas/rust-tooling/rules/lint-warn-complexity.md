---
title: Enable clippy::complexity to Flag Unnecessarily Convoluted Code
impact: MEDIUM
impactDescription: Replaces manual reimplementations of standard combinators with the direct call
tags: [tooling, clippy, complexity, readability]
---

# Enable clippy::complexity to Flag Unnecessarily Convoluted Code [MEDIUM]

## Description
`clippy::complexity` targets code that works correctly but does something the standard library or a simpler expression already does more directly — a hand-written `match` that's exactly `Option::map`, a `filter().count()` chain that could be one call, a boolean double-negation that could just be the plain condition. None of this is a bug, but each instance adds a small amount of extra reasoning a reader has to do to confirm the convoluted version is equivalent to the simple one, and that cost compounds across a codebase. The lint's suggestions are close to mechanical — most fixes are a direct substitution — which makes this one of the cheaper clippy groups to enable and actually clear out.

## Bad Example
```rust
// Manually reimplements Option::map with a match.
let result = match option {
    Some(x) => Some(x + 1),
    None => None,
};
```

## Good Example
```rust
let result = option.map(|x| x + 1);
```

## Notes
- `bind_instead_of_map` (an `and_then(|x| Some(...))` that should just be `map`), `clone_on_copy` (a `.clone()` call on a `Copy` type that could be a plain assignment), and `useless_let_if_seq` (an uninitialized `let` followed by an if/else that assigns it, instead of `let x = if cond { .. } else { .. }`) are three of the most common real-world hits from this group.
- A chain like `nums.iter().cloned().filter(|x| *x > 0).collect::<Vec<_>>()` followed by manual indexing to grab the first match is exactly what `.find(...)` already expresses in one call — complexity warnings often point at a missing iterator adapter rather than a logic error.
- Enable this alongside `clippy::style`; the two groups frequently flag adjacent code (a style lint about `is_empty()` next to a complexity lint about a redundant `match`), and fixing both together tends to be faster than two separate passes.
- Treat a high volume of complexity warnings on first enabling the lint as a sign the codebase leans on manual control flow where iterator combinators would read more directly — it's worth fixing incrementally rather than mass-suppressing with a blanket `#[allow]`.

## References
- [lint-warn-style](lint-warn-style.md)
- [lint-warn-perf](lint-warn-perf.md)
- [lint-pedantic-selective](lint-pedantic-selective.md)
