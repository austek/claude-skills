---
title: Drop the get_ Prefix on Plain Field Accessors
impact: MEDIUM
impactDescription: Matches std's accessor style (len(), not get_len())
tags: [naming, api-design, getters, style]
---

# Drop the get_ Prefix on Plain Field Accessors [MEDIUM]

## Description
Rust's convention is the opposite of the getter-prefix habit common in other languages: a method that just hands back a field, with no extra computation or fallibility, drops the `get_` prefix entirely — `len()`, `name()`, `value()`, not `get_len()`, `get_name()`, `get_value()`. The `get` prefix is reserved for a narrower case: a lookup that can fail or do real work, which is exactly what `Vec::get(index) -> Option<&T>` and `HashMap::get(key) -> Option<&V>` do — both return `Option` because the requested item might not be there. Applying `get_` to a plain accessor blurs that distinction and adds four characters of pure noise to every call site.

## Bad Example
```rust
struct Invoice {
    total: f64,
    paid: bool,
}

impl Invoice {
    fn get_total(&self) -> f64 {   // plain field access, no get_ needed
        self.total
    }

    fn get_is_paid(&self) -> bool { // doubly wrong: also needs is_, not get_is_
        self.paid
    }
}
```

## Good Example
```rust
struct Invoice {
    total: f64,
    paid: bool,
}

impl Invoice {
    fn total(&self) -> f64 {
        self.total
    }

    fn is_paid(&self) -> bool {
        self.paid
    }
}

impl InvoiceStore {
    // get_ is earned here: the lookup can fail.
    fn get(&self, id: InvoiceId) -> Option<&Invoice> { ... }
}
```

## Notes
- Setters keep an explicit `set_` prefix even though getters drop `get_` — the asymmetry is intentional, since `fn total(&mut self, v: f64)` reads ambiguously as either a getter or setter, while `set_total` doesn't.
- Builder methods also skip both prefixes: `ConfigBuilder::timeout(mut self, d: Duration) -> Self` reads naturally in a chained call even though it's conceptually closer to a setter.
- `get_mut` is the accepted exception for the mutable counterpart of a fallible `get` — `Vec::get_mut`, `HashMap::get_mut` — because it's still describing the same fallible-lookup family, just returning `&mut T`.

## References
- [name-is-has-bool](name-is-has-bool.md)
