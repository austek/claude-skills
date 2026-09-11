---
title: Require the Least Restrictive Fn Trait a Callback Needs
impact: MEDIUM
impactDescription: Accepts the widest set of caller closures, including move-consuming ones
tags: [closures, traits, api-design]
---

# Require the Least Restrictive Fn Trait a Callback Needs [MEDIUM]

## Description
The three closure traits form a hierarchy by how much they demand of the closures that satisfy them: `FnOnce` is the loosest, satisfied by literally every closure since it only promises the closure is callable at least once, even one that consumes and drops its own captures in the process; `FnMut` sits in the middle, requiring repeatable calls and allowing captures to be mutated between them; `Fn` is the tightest, requiring both repeatability and read-only access to captures, which is what makes a `Fn` closure safe to call from multiple places at once. A function accepting a callback should bound it with the loosest of these three traits its own body can get away with, because the bound directly determines which closures are legal arguments — asking for `Fn` when the function only ever calls the callback once shuts out any closure that needs to consume something it captured, even though nothing about calling it once actually required that restriction.

## Bad Example
```rust
// F: Fn is too strict — the closure is only called once here,
// so a move-consuming closure is unnecessarily rejected
fn run_once_bad<F: Fn() -> String>(f: F) -> String {
    f()
}

fn demo() {
    let s = String::from("hello");
    // run_once_bad(move || s) // compile error: `s` moved into a closure that only implements FnOnce
    let _ = run_once_bad(|| String::from("ok")); // forced into a non-consuming closure
}
```

## Good Example
```rust
// Call exactly once — accept FnOnce, the widest possible bound
fn run_once<F: FnOnce() -> String>(f: F) -> String {
    f()
}

// Call multiple times, closure may mutate its captures — accept FnMut
fn retry<F: FnMut() -> bool>(mut f: F, attempts: usize) -> bool {
    (0..attempts).any(|_| f())
}

// Call multiple times, need it shareable/re-entrant — accept Fn
fn for_each<T, F: Fn(&T)>(items: &[T], f: F) {
    for item in items {
        f(item);
    }
}

fn demo() {
    let s = String::from("hello");
    let result = run_once(move || s.to_uppercase()); // move-consuming closure now accepted
    assert_eq!(result, "HELLO");
}
```

## Notes
- The supertrait relationship runs one way only — every `Fn` closure is also a valid `FnMut` and `FnOnce`, but a closure that only implements `FnOnce` can't satisfy an `FnMut` or `Fn` bound. Picking the loosest bound the implementation can live with is therefore what maximizes the range of closures a caller is allowed to pass in.
- Even though nothing in the trait's name says so, a variable or parameter holding an `FnMut` closure has to be declared `mut` wherever it's actually invoked, since calling it requires a mutable borrow of the closure itself.
- The standard library follows this same rule internally: `Iterator::map` only needs `FnMut` because it calls the closure once per item and may need to update state between calls, while `thread::spawn` demands the stricter `FnOnce + Send + 'static` because the closure runs exactly once, on a different thread, and needs to not reference anything that could go away before it runs.

## References
- [closure-move-capture](closure-move-capture.md)
- [closure-static-vs-dyn](closure-static-vs-dyn.md)
