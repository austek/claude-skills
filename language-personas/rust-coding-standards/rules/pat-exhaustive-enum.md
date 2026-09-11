---
title: Match Owned Enums Exhaustively, Avoid Catch-All Wildcards
impact: HIGH
impactDescription: Turns a missed new variant into a compile error instead of a silent logic bug
tags: [pattern-matching, enums, exhaustiveness, correctness]
---

# Match Owned Enums Exhaustively, Avoid Catch-All Wildcards [HIGH]

## Description
Adding a `_ =>` arm to a match over an enum feels harmless, but it quietly changes what happens the next time someone adds a variant to that enum: instead of the compiler stopping the build at every place the new variant needs handling, the wildcard swallows it and the match keeps compiling as if nothing changed. Whatever behavior the wildcard arm implements now applies to a variant its author never actually considered. Writing out every variant by name flips that outcome — the compiler becomes an enforced checklist, and a new variant fails every match that hasn't been updated to account for it, which is precisely the signal a maintainer wants. The one legitimate exception is a foreign enum marked `#[non_exhaustive]`, where the language itself won't let a match compile without a catch-all; outside of that case, a wildcard on an enum a project owns is worth treating as a bug waiting to happen the moment that enum grows.

## Bad Example
```rust
#[derive(Debug)]
enum Status {
    Active,
    Pending,
    Closed,
}

fn describe(s: &Status) -> &'static str {
    match s {
        Status::Active => "active",
        _ => "inactive", // hides Status::Pending; a future Status::Suspended goes unnoticed
    }
}
// If Status::Suspended is added later, this still compiles and silently
// returns "inactive" for it — a logic bug the compiler never catches.
```

## Good Example
```rust
#[derive(Debug)]
enum Status {
    Active,
    Pending,
    Closed,
}

fn describe(s: &Status) -> &'static str {
    match s {
        Status::Active => "active",
        Status::Pending => "pending",
        Status::Closed => "closed",
        // Adding Status::Suspended now causes a compile error here — intended.
    }
}

// Group variants that share handling with `|` instead of falling back to `_`
fn is_terminal(s: &Status) -> bool {
    match s {
        Status::Closed | Status::Pending => true,
        Status::Active => false,
    }
}
```

## Notes
- `#[non_exhaustive]` on an enum from an external crate is a promise from that crate's author that more variants may appear in a future release without it counting as a breaking change — the compiler responds by refusing to compile a match without a wildcard, since it genuinely cannot know every variant that might exist later. A short comment on the arm explaining that the wildcard is required, not chosen, helps the next reader distinguish this case from a lazy catch-all.
- The `clippy::wildcard_enum_match_arm` lint, part of `clippy::restriction`, specifically flags a wildcard on an enum that isn't `#[non_exhaustive]` and could instead list its variants explicitly — worth enabling on any project that wants this rule enforced automatically rather than caught in review.
- When several variants genuinely share identical handling, `|` inside a single arm keeps the match exhaustive while still avoiding repetition — it's the tool to reach for instead of `_` whenever the set of variants that share a branch is fixed and already known, rather than "whatever's left."

## References
- [api-non-exhaustive](api-non-exhaustive.md)
- [type-enum-states](type-enum-states.md)
- [pat-matches-macro](pat-matches-macro.md)
