---
title: Notice format! Allocating Inside a Loop
impact: LOW
impactDescription: An easy-to-miss allocation that repeats on every iteration
tags: [anti-pattern, performance, allocation, code-smell]
---

# Notice format! Allocating Inside a Loop [LOW]

## Description
`format!` reads naturally and is the right tool for a one-off string, which is exactly why it slips unnoticed into a loop body — nothing about the call site looks different whether it runs once or a million times, but each call allocates a fresh `String` regardless. The pattern to watch for in review is a `format!` (or repeated `+`/`push_str` building a new `String` each pass) sitting inside a `for` loop or a function called at high frequency; the fix is almost always mechanical once spotted — hoist a reusable buffer out of the loop and `write!` into it after a `.clear()`.

## Bad Example
```rust
fn log_events(events: &[Event]) {
    for event in events {
        // Allocates a new String on every single iteration.
        let line = format!("[{}] {}: {}", event.level, event.source, event.message);
        logger.log(&line);
    }
}
```

## Good Example
```rust
use std::fmt::Write;

fn log_events(events: &[Event]) {
    let mut line = String::with_capacity(256);
    for event in events {
        line.clear(); // keeps the allocation, drops the contents
        write!(line, "[{}] {}: {}", event.level, event.source, event.message).unwrap();
        logger.log(&line);
    }
}
```

## Notes
- Not every `format!` in a function that happens to be called often is worth chasing — the ones that matter are inside the loop or on the path that runs per-item, not the one-time setup surrounding it.
- `clippy::format_in_format_args` catches a specific, narrower case of nested `format!` calls, but doesn't flag `format!` inside a hot loop generally — this pattern mostly needs a human eye or a profiler pointing at the allocation.
- Error paths and one-off diagnostic output (`println!("Debug: {:?}", value)`, a `format!` building a startup config path) are exactly where `format!`'s convenience is worth keeping — this is a hot-path concern, not a blanket rule against the macro.

## References
- [mem-avoid-format](mem-avoid-format.md)
- [mem-write-over-format](mem-write-over-format.md)
- [mem-reuse-collections](mem-reuse-collections.md)
