---
title: Return Closures as impl Fn, Not Box dyn Fn
impact: MEDIUM
impactDescription: Avoids a heap allocation and a vtable call on every invocation
tags: [closures, zero-cost, api-design]
---

# Return Closures as impl Fn, Not Box dyn Fn [MEDIUM]

## Description
Every closure expression in Rust has its own unique, compiler-generated type that has no name a programmer could write out — which is exactly the problem `impl Trait` in return position was designed to solve, letting a function return "some type that implements `Fn(i32) -> i32`" without ever needing to spell out what that type actually is. Because the real concrete type is still known to the compiler, calling the returned closure dispatches statically and needs no heap allocation at all. `Box<dyn Fn>` throws that information away deliberately — it erases the concrete type behind a trait object, which costs an allocation when the box is built and a vtable indirection on every call afterward. That cost buys something real, but only in specific situations: when a function's different branches would otherwise need to return different concrete closure types that `impl Trait` can't unify into one return type, or when a closure needs to sit in a struct field or a collection next to other, differently-shaped closures.

## Bad Example
```rust
// Allocates on the heap for no benefit — there's only one concrete closure type here
fn adder_bad(n: i32) -> Box<dyn Fn(i32) -> i32> {
    Box::new(move |x| x + n)
}
```

## Good Example
```rust
// Zero allocation, statically dispatched — the compiler can inline this
fn adder(n: i32) -> impl Fn(i32) -> i32 {
    move |x| x + n
}

fn apply(f: impl Fn(i32) -> i32, value: i32) -> i32 {
    f(value)
}

fn demo() {
    let add5 = adder(5);
    assert_eq!(apply(add5, 10), 15);
}

// Boxing is genuinely required when branches return different concrete closure types
fn make_transform(double: bool) -> Box<dyn Fn(i32) -> i32> {
    if double {
        Box::new(|x| x * 2)   // one concrete closure type
    } else {
        Box::new(|x| x + 100) // a different concrete type — impl Fn cannot unify these
    }
}
```

## Notes
- Returning `impl FnMut` doesn't change the fact that calling an `FnMut` closure needs a mutable borrow of it — the caller still has to bind the result to a `mut` variable, even though nothing about the return type's syntax hints at that requirement.
- A `Vec<impl Fn>` can't exist, since `impl Trait` names one specific concrete type per use and a vector needs every element to share one type — anything that wants to collect closures of genuinely different shapes, like a registry of event handlers, needs the uniform representation `Box<dyn Fn>` provides.
- A method that internally boxes a closure for storage doesn't have to expose that detail in its own signature — `fn add_step(&mut self, f: impl Fn(i32) -> i32 + 'static)` can stay generic at the API boundary even while its body wraps the argument in a `Box` before pushing it into a `Vec<Box<dyn Fn(i32) -> i32>>`.

## References
- [closure-static-vs-dyn](closure-static-vs-dyn.md)
- [closure-fn-trait-bounds](closure-fn-trait-bounds.md)
