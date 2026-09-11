---
title: Rely on macro_rules Hygiene, Use $crate for Item Paths
impact: HIGH
impactDescription: Prevents an exported macro from silently breaking when called from another crate
tags: [macros, declarative-macros, correctness]
---

# Rely on macro_rules Hygiene, Use $crate for Item Paths [HIGH]

## Description
Hygiene is the property that keeps a `let` binding or loop label a macro introduces from colliding with anything of the same name already in scope where the macro is invoked — each expansion effectively gets its own private namespace for names it introduces itself. That protection, though, doesn't extend to paths pointing at items: a macro body that writes `crate::helper()` isn't specifying "the crate that defined this macro," it's specifying "whichever crate the macro happens to be expanded in," because `crate::` is resolved fresh at every expansion site. That's invisible while testing the macro inside its own defining crate, since there `crate::` happens to point at the right place by coincidence — and it breaks the instant the macro is exported and called from anywhere else. `$crate::helper()` fixes this by always resolving to the defining crate specifically, independent of where the macro is actually invoked from.

## Bad Example
```rust
// lib.rs
pub fn log_value(v: &str) {
    println!("[log] {v}");
}

#[macro_export]
macro_rules! log {
    ($val:expr) => {
        // WRONG: resolves relative to the caller's crate, not the defining crate
        crate::log_value(&format!("{:?}", $val));
    };
}
```
```rust
// consumer/src/main.rs
use mylib::log;

fn main() {
    log!(42); // compile error: `crate::log_value` not found in consumer
}
```

## Good Example
```rust
// lib.rs
pub fn log_value(v: &str) {
    println!("[log] {v}");
}

#[macro_export]
macro_rules! log {
    ($val:expr) => {
        // $crate always expands to the crate that defined this macro
        $crate::log_value(&format!("{:?}", $val));
    };
}

// Hygiene for local bindings: `tmp` inside the macro never clashes with the caller's `tmp`
macro_rules! swap {
    ($a:expr, $b:expr) => {{
        let tmp = $a;
        $a = $b;
        $b = tmp;
    }};
}

fn main() {
    let tmp = "outer"; // unrelated to the macro's internal `tmp`
    let (mut x, mut y) = (1, 2);
    swap!(x, y);
    assert_eq!((x, y), (2, 1));
    assert_eq!(tmp, "outer");
}
```

## Notes
- Treat `$crate::` as the default for any path a macro needs to any item in its own defining crate — a bare `crate::` path only happens to work when the macro is expanded inside its own crate, and silently stops working the moment it's exported and used from elsewhere, including through a re-export.
- Hygiene only covers names the macro itself introduces — an identifier passed into the macro as `$name:ident` still comes from the caller's own scope, on purpose, and a macro that binds to or shadows that identifier is doing so deliberately, not accidentally, since hygiene offers no protection there.
- Combining `$crate::`-anchored paths with a `#[doc(hidden)] pub mod __private` for any helper a macro's expansion needs keeps the generated code working correctly from any call site without turning those helpers into part of the crate's documented public API.

## References
- [macro-export-crate-path](macro-export-crate-path.md)
- [macro-private-helpers](macro-private-helpers.md)
- [macro-prefer-functions](macro-prefer-functions.md)
