---
title: Capture Only What You Use with Disjoint Closure Captures
impact: MEDIUM
impactDescription: Removes spurious borrow conflicts between sibling struct fields
tags: [closures, ownership, edition-2021]
---

# Capture Only What You Use with Disjoint Closure Captures [MEDIUM]

## Description
Up through the 2018 edition, a closure's capture analysis operated on whole variable names — a closure body that only reads `config.threshold` still counted, as far as the borrow checker was concerned, as capturing all of `config`, which meant nothing else could touch `config.label` while that closure was alive. The 2021 edition changed the granularity of that analysis so a closure captures only the specific field path it actually touches, leaving every other field of the same struct free for other code to use at the same time. That precision has a boundary, though: `move` still moves whatever place it's attached to as a whole, so `move || config.threshold > 0` moves the entirety of `config` into the closure even though the body only reads one field of it. Getting `move` to affect just the field it's used on means pulling that field into a local variable first and moving the local instead of the struct.

## Bad Example
```rust
struct Config {
    threshold: i32,
    label: String,
}

fn make_checker(config: Config) -> (impl Fn() -> bool, String) {
    // move captures the WHOLE `config`, even though only `threshold` is used —
    // config.label becomes unusable after this line
    let checker = move || config.threshold > 0;
    (checker, config.label) // error: use of moved value `config`
}
```

## Good Example
```rust
struct Config {
    threshold: i32,
    label: String,
}

fn demo_borrow() {
    let config = Config { threshold: 10, label: String::from("active") };

    // Edition 2021: this closure captures only `config.threshold`.
    // `config.label` is NOT captured and stays accessible.
    let check = || config.threshold > 0;
    println!("label: {}", config.label); // fine — not captured by `check`
    assert!(check());
}

fn make_checker(config: Config) -> (impl Fn() -> bool, String) {
    // Bind the field to a local first, then move only that local (a Copy i32)
    let threshold = config.threshold;
    let checker = move || threshold > 0;
    (checker, config.label) // config.label is still available
}
```

## Notes
- What actually changed in 2021 is granularity: the compiler now tracks `foo.bar` as a capture distinct from `foo` itself, rather than always rounding up to the containing variable — crates still targeting the 2018 edition need the local-variable workaround even for closures that only borrow.
- A field of a `Copy` type — an integer or a boolean, say — is copied into a `move` closure rather than moved, so the source struct keeps that field usable afterward, provided it's the `Copy` field itself being captured and not something that pulls in the struct around it.
- Start every closure as a plain borrow and only add `move` once there's an actual reason to — typically because the closure needs to outlive the scope it was created in — rather than reaching for `move` by default.

## References
- [closure-move-capture](closure-move-capture.md)
- [own-borrow-over-clone](own-borrow-over-clone.md)
