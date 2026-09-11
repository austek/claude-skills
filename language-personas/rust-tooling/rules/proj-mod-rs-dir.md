---
title: Choose mod.rs or Adjacent-File Style Consistently
impact: LOW
impactDescription: Removes the per-directory guesswork of which multi-file-module style is in play
tags: [tooling, project-structure, modules, rust-2018]
---

# Choose mod.rs or Adjacent-File Style Consistently [LOW]

## Description
Rust supports two equivalent layouts for a module that spans multiple files: put a `mod.rs` inside a directory named after the module (`user/mod.rs`, `user/model.rs`), or put a same-named file next to a directory holding its submodules (`user.rs` alongside a `user/` directory containing `model.rs`). Both compile identically — `mod user;` looks for either `user.rs` or `user/mod.rs` — so the only real question is which one a given project uses consistently. `mod.rs` reads clearly as "this whole directory is one module" and scales well once a module has many submodules; the adjacent-file style lets you see a module's own top-level code without opening its directory first, and is the form the Rust 2018+ default lint configuration nudges toward for simpler modules. The choice matters less than picking one and sticking to it across the whole codebase.

## Bad Example
```
src/
├── user/
│   ├── mod.rs
│   └── model.rs
├── order.rs          # a different style used for the very next module
└── order/
    └── model.rs
```

## Good Example
```
src/
├── user/
│   ├── mod.rs          # module root
│   ├── model.rs
│   └── repository.rs
├── order/
│   ├── mod.rs           # same style used everywhere
│   └── model.rs
└── lib.rs
```

## Notes
- A rough rule of thumb: adjacent-file style (`user.rs` + `user/`) tends to read better for a module with only a few submodules, while `mod.rs` scales more comfortably once a module accumulates many of them — large projects like tokio and serde lean on the `mod.rs` convention throughout.
- Clippy can enforce whichever style you pick automatically: `mod_module_files = "warn"` flags adjacent-file modules (pushing toward `mod.rs`), and `self_named_module_files = "warn"` does the reverse — enable exactly one, never both, since they contradict each other.
- Whichever style is chosen, keep `mod.rs` (or the adjacent root file) itself thin — its job is declaring submodules and re-exporting the module's public surface, not holding substantial logic of its own.
- Mixing styles within one codebase is the actual problem worth avoiding — a reader who has to check, per directory, which convention is in play loses exactly the navigational clarity either style is supposed to provide on its own.

## References
- [proj-flat-small](proj-flat-small.md)
- [proj-mod-by-feature](proj-mod-by-feature.md)
- [proj-pub-use-reexport](proj-pub-use-reexport.md)
