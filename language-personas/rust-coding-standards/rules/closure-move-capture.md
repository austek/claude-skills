---
title: Use move for Closures That Outlive the Current Scope
impact: HIGH
impactDescription: Fixes lifetime errors when a closure escapes to a thread, task, or return value
tags: [closures, ownership, lifetimes]
---

# Use move for Closures That Outlive the Current Scope [HIGH]

## Description
A closure that only borrows the variables it captures is tied to the lifetime of whatever it borrowed from — it cannot legally exist any longer than the data it's referencing does. That becomes a hard constraint the moment a closure needs to escape its creating scope: get passed into a spawned thread, stashed inside a long-lived struct, or handed back as a function's return value, and it typically has to satisfy `'static`, meaning nothing it holds can be a borrow tied to a scope that might end. `move` is what makes that possible — instead of capturing references, it moves ownership of the captured variables directly into the closure, so the closure is now self-sufficient and independent of the scope where it was written. If the code around the closure still needs the original value afterward, the fix is to clone the value before the `move` and hand the closure the clone, keeping the original untouched.

## Bad Example
```rust
fn spawn_bad() {
    let data = vec![1, 2, 3];
    // Borrowing `data` across a thread boundary is rejected —
    // the closure could outlive this stack frame
    std::thread::spawn(|| println!("{data:?}")); // error: borrowed value does not live long enough
}
```

## Good Example
```rust
fn process(data: &[i32]) -> i32 {
    data.iter().sum()
}

// Return a closure that owns its capture via `move`
fn make_greeter(name: String) -> impl Fn() {
    move || println!("hello, {name}")
}

// Clone before `move` when the value is still needed in both places
fn spawn_and_keep(data: Vec<i32>) -> std::thread::JoinHandle<i32> {
    let data_for_thread = data.clone(); // the clone goes into the closure
    let handle = std::thread::spawn(move || process(&data_for_thread));
    println!("original still owned: {data:?}"); // `data` remains available here
    handle
}
```

## Notes
- `std::thread::spawn` specifically requires its closure argument to be `'static`, and `move` is what makes that achievable, as long as every value the closure captures is itself both `'static` and `Send`.
- Cloning the entire struct a field lives in just to get `move` to work is usually more than necessary — pairing this with disjoint field capture means `move` can transfer only the specific field a closure actually needs, leaving the rest of the struct untouched.
- The same logic carries over to async code: `tokio::spawn(async move { ... })` takes ownership of whatever the block captures, so any value still needed outside the block should be cloned before entering it, not after.
- `move` decides how a closure acquires its captures, not what the closure is allowed to do with them afterward — a closure created with `move` can still end up implementing `Fn` or `FnMut`, depending entirely on whether its body reads, mutates, or consumes what it captured.

## References
- [closure-disjoint-capture](closure-disjoint-capture.md)
- [async-clone-before-await](async-clone-before-await.md)
- [own-move-large](own-move-large.md)
