---
title: Choose Static or Dynamic Dispatch Deliberately
impact: HIGH
impactDescription: Wrong default either leaves inlining on the table or blocks heterogeneous storage
tags: [trait-design, dynamic-dispatch, generics, performance]
---

# Choose Static or Dynamic Dispatch Deliberately [HIGH]

## Description
Generic bounds and `impl Trait` monomorphize at compile time: each concrete type gets its own specialized copy the compiler can inline and optimize, at the cost of binary size growing with every distinct instantiation. `dyn Trait` stores a fat pointer (data plus vtable) and dispatches at runtime through one code path, at the cost of an indirection per call — but it's the only option for storing heterogeneous types in one collection or accepting a plug-in registered at runtime. Reaching for `dyn` "to be flexible" everywhere blocks inlining and forces heap allocation even when only one concrete type is ever used; reaching for generics when the collection is genuinely heterogeneous simply doesn't compile.

## Bad Example
```rust
trait Shape { fn area(&self) -> f64; }

struct Circle { radius: f64 }
impl Shape for Circle {
    fn area(&self) -> f64 { std::f64::consts::PI * self.radius * self.radius }
}

// Unnecessary boxing and dispatch overhead when only one concrete type is used.
fn total_area(shapes: &[Box<dyn Shape>]) -> f64 {
    shapes.iter().map(|s| s.area()).sum()
}
```

## Good Example
```rust
trait Shape { fn area(&self) -> f64; }

// Static dispatch: monomorphized, the compiler can inline area().
fn total_area_generic<S: Shape>(shapes: &[S]) -> f64 {
    shapes.iter().map(|s| s.area()).sum()
}

// Dynamic dispatch: reach for it only when the collection is genuinely
// heterogeneous, e.g. Vec<Box<dyn Shape>> mixing Circle and Rect.
fn total_area_dyn(shapes: &[Box<dyn Shape>]) -> f64 {
    shapes.iter().map(|s| s.area()).sum()
}
```

## Notes
- Default to generics or `impl Trait` for a single known concrete type or a hot path where inlining matters.
- Reach for `dyn Trait` for heterogeneous collections, trait objects stored across calls, or a plug-in/callback registered at runtime.
- `dyn Trait` also caps binary size growth when a generic function would otherwise be monomorphized over many call-site types.
- `dyn Trait` requires the trait to be object-safe — a generic method or an associated constant on the trait blocks this path entirely.

## References
- [type-generic-bounds](type-generic-bounds.md)
- [trait-object-safety](trait-object-safety.md)
