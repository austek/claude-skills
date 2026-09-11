---
title: Use let else for Early-Return Pattern Extraction
impact: MEDIUM
impactDescription: Eliminates rightward drift from stacked if let blocks
tags: [pattern-matching, readability, control-flow]
---

# Use let else for Early-Return Pattern Extraction [MEDIUM]

## Description
`let ... else`, available since Rust 1.65, splits a pattern match into two clearly separated outcomes: if the pattern matches, the bound variable becomes available in the rest of the enclosing scope, and if it doesn't, the `else` block runs instead — and that block is required to leave the scope entirely, whether by returning, breaking, continuing, or panicking. That requirement is what makes the construct useful for flattening code: because the failure path is guaranteed to exit, everything after the `let` statement can assume the pattern matched, with no extra nesting needed to express that assumption. Compare that to stacking several `if let` blocks to extract several values in sequence, where the actual success-path logic ends up buried one indentation level deeper for every value it depends on.

## Bad Example
```rust
fn process(input: Option<String>) -> Option<u32> {
    if let Some(s) = input {
        if let Ok(n) = s.trim().parse::<u32>() {
            if n > 0 {
                return Some(n * 2);
            } else {
                return None;
            }
        } else {
            return None;
        }
    } else {
        return None;
    }
}
```

## Good Example
```rust
fn process(input: Option<String>) -> Option<u32> {
    let Some(s) = input else { return None; };
    let Ok(n) = s.trim().parse::<u32>() else { return None; };
    if n == 0 {
        return None;
    }
    Some(n * 2)
}

// The `else` block can use any diverging expression, including error macros
use anyhow::{bail, Result};
use std::collections::HashMap;

fn get_id(map: &HashMap<String, u64>, key: &str) -> Result<u64> {
    let Some(&id) = map.get(key) else {
        bail!("key '{}' not found", key);
    };
    Ok(id)
}
```

## Notes
- Scoping only goes one direction here: the bound name becomes available after the `let ... else` statement, and the `else` block itself has no access to it — by the time `else` runs, the pattern is known not to have matched, so there's nothing to bind.
- When the failure path is nothing more than propagating an error upward unchanged, `?` is the more direct tool; `let ... else` earns its keep specifically when the failure needs to `return`, `break`, or `continue` — control flow `?` has no way to express.
- Clippy ships a lint, `clippy::manual_let_else`, that specifically looks for nested `if let` chains a codebase could flatten into this form, which makes migrating existing code mostly automatic.

## References
- [err-question-mark](err-question-mark.md)
- [pat-exhaustive-enum](pat-exhaustive-enum.md)
