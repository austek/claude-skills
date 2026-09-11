---
title: Choose Generic impl Fn or dyn Fn by Call Site, Not by Habit
impact: MEDIUM
impactDescription: Trades binary size against inlining based on the callback's actual usage pattern
tags: [closures, dispatch, api-design]
---

# Choose Generic impl Fn or dyn Fn by Call Site, Not by Habit [MEDIUM]

## Description
A function generic over a closure type — written `F: Fn(...) -> ...` or `impl Fn` — gets a separately compiled, specialized version of itself for every distinct closure type that's ever substituted in at a call site, which is what allows the compiler to inline the closure's body right into the function and dispatch with no indirection at all. That specialization isn't free, though: a function called with ten different closures throughout a codebase ends up compiled ten separate times, which adds up in binary size. `&dyn Fn` and `Box<dyn Fn>` take the opposite trade — a single compiled function body handles every closure via a vtable lookup at each call, which keeps code size down and, unlike the generic form, actually allows storing closures of different underlying types together, such as a list of registered event handlers. Neither approach is universally correct; the right one falls out of how a given callback is actually used, not a general rule of thumb.

## Bad Example
```rust
// A struct that wants to hold ANY closure but is generic over one concrete type —
// this can only ever hold a single closure type, defeating the purpose of a registry
struct BadRegistry<F: Fn(&str)> {
    handler: F,
}

// Using Box<dyn Fn> on a hot, single-call-site inner loop pays a vtable cost for no benefit
fn transform_slow(xs: &[i32], f: &dyn Fn(i32) -> i32) -> Vec<i32> {
    xs.iter().map(|&x| f(x)).collect()
}
```

## Good Example
```rust
// Generic / static dispatch — preferred for hot paths: inlinable, zero allocation
fn transform<F: Fn(i32) -> i32>(xs: &[i32], f: F) -> Vec<i32> {
    xs.iter().map(|&x| f(x)).collect()
}

// Dynamic dispatch — required to store heterogeneous closures
struct Registry {
    handlers: Vec<Box<dyn Fn(&str)>>,
}

impl Registry {
    fn register(&mut self, handler: impl Fn(&str) + 'static) {
        self.handlers.push(Box::new(handler));
    }

    fn dispatch(&self, event: &str) {
        for handler in &self.handlers {
            handler(event);
        }
    }
}
```

## Notes
- As a starting point for the decision: a hot loop with one call site fits a generic `F: Fn`/`impl Fn`; a callback that lives in a struct field or a collection alongside other closures needs `Box<dyn Fn>`; a closure that's only ever borrowed once and never stored fits `&dyn Fn`, which sidesteps an allocation without requiring the caller to give up ownership; a closure that needs to be held across an `await` point typically needs `Box<dyn Fn + Send>`.
- `&dyn Fn` earns its place specifically in the borrow-once, don't-store case — passing `&closure` avoids the allocation a `Box` would introduce, while still giving the callee a uniform type to call regardless of the closure's actual concrete type.
- Monomorphization isn't automatically the cheaper option — a generic function substituted with many distinct closure types compiles to many distinct copies, and in aggregate that can outweigh what a single `dyn Fn` vtable would have cost; the honest way to decide is to measure, not assume.

## References
- [closure-fn-trait-bounds](closure-fn-trait-bounds.md)
- [trait-dyn-vs-generic](trait-dyn-vs-generic.md)
- [type-generic-bounds](type-generic-bounds.md)
