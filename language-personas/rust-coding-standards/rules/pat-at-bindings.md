---
title: Use @ Bindings to Capture While Matching
impact: LOW
impactDescription: Collapses a guard-and-rebind pair into one readable pattern arm
tags: [pattern-matching, readability]
---

# Use @ Bindings to Capture While Matching [LOW]

## Description
Writing `name @ pattern` inside a match arm does two things at once: it checks the scrutinee against `pattern`, and it gives the whole matched value a name usable in that arm's body. Without it, a developer facing the same need typically reaches for a separate match guard that repeats the same condition already implied by the pattern, then re-reads the original value to get at it — two things doing the work of one. Reserving `@` for exactly this situation keeps the constraint and the name it produces sitting next to each other in the source, which pays off most clearly on a range pattern or a nested destructure, where the value being tested and the value the arm body actually wants to use are the same thing.

## Bad Example
```rust
#[derive(Debug)]
enum Command {
    Move { x: i32, y: i32 },
}

fn validate_move(cmd: &Command) {
    match cmd {
        // x is already bound by the destructure, but the range check is a separate
        // guard clause that duplicates what the pattern itself could express
        Command::Move { x, y } if *x >= 0 && *x <= 100 => {
            println!("valid move to x={x}, y={y}");
        }
        _ => println!("invalid command"),
    }
}
```

## Good Example
```rust
#[derive(Debug)]
enum Command {
    Move { x: i32, y: i32 },
}

fn validate_move(cmd: &Command) {
    match cmd {
        // Destructures x, checks it falls in 0..=100, and binds it to x_pos — one expression
        Command::Move { x: x_pos @ 0..=100, y } => {
            println!("valid move to x={x_pos}, y={y}");
        }
        _ => println!("invalid command"),
    }
}

fn classify(n: u32) -> String {
    match n {
        id @ 1..=9 => format!("single digit: {id}"),
        id @ 10..=99 => format!("two digits: {id}"),
        _ => String::from("large"),
    }
}
```

## Notes
- This isn't limited to `match` — the same `@` syntax is legal anywhere a pattern is, including `if let`, `while let`, `let ... else`, and even a function parameter's pattern.
- One particularly useful form binds an entire enum variant while a guard peers into its contents, as in `whole @ Packet::Data(bytes) if !bytes.is_empty()` — the code gets both the untouched original value for logging and the ability to inspect its payload, with no need to reconstruct the variant afterward.
- Treat a match guard that just re-checks something the pattern could already express as a signal to reach for `@` instead — it's usually a sign the guard is compensating for a binding the pattern itself should be producing.

## References
- [pat-exhaustive-enum](pat-exhaustive-enum.md)
- [type-enum-states](type-enum-states.md)
