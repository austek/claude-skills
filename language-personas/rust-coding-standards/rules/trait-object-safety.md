---
title: Keep a Trait Object-Safe When It Needs to Support dyn
impact: MEDIUM
impactDescription: Avoids a hard compiler error at the dyn use site, far from the trait definition
tags: [trait-design, object-safety, dyn-trait, vtable]
---

# Keep a Trait Object-Safe When It Needs to Support dyn [MEDIUM]

## Description
Only object-safe (dyn-compatible) traits can be used as `dyn Trait`: every method must be dispatchable through a vtable, which rules out generic type parameters on methods, returning `Self` by value, and associated constants. Violating this produces a hard compiler error (E0038) — but only at the point where someone tries to use the trait as `dyn Trait`, which can be far from where the offending method was originally added. When a trait genuinely needs both a generic convenience method and `dyn`-compatibility, gate the non-dispatchable method with `where Self: Sized` — that excludes it from the vtable while leaving the rest of the trait object-safe.

## Bad Example
```rust
trait Transformer {
    // Generic method -- not dispatchable, makes the whole trait non-object-safe.
    fn transform<T: std::fmt::Debug>(&self, value: T) -> String;
    fn name(&self) -> &str;
}

// error[E0038]: the trait `Transformer` cannot be made into an object
// fn apply(t: &dyn Transformer, x: i32) { ... }
```

## Good Example
```rust
trait Transformer {
    // Core dispatchable method -- always in the vtable.
    fn transform_str(&self, value: &str) -> String;
    fn name(&self) -> &str;

    // Generic convenience method excluded from the vtable, still usable
    // through a concrete type via static dispatch.
    fn transform_debug<T: std::fmt::Debug>(&self, value: T) -> String
    where
        Self: Sized,
    {
        self.transform_str(&format!("{value:?}"))
    }
}
```

## Notes
- Allowed in `dyn Trait`: `&self`/`&mut self` methods, associated types (erased but fixed per impl), and any method gated `where Self: Sized`.
- Not allowed: methods returning `Self` by value (use `Box<Self>` or gate with `Sized`), generic method parameters, associated constants.
- A `where Self: Sized` method is simply excluded from the vtable — callers through `dyn Trait` never see it, but callers with a concrete type still can.
- Check object-safety early, at the trait definition, rather than discovering the violation later at an unrelated `dyn` call site.

## References
- [trait-dyn-vs-generic](trait-dyn-vs-generic.md)
- [api-sealed-trait](api-sealed-trait.md)
