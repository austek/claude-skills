---
title: Implement Deref Only for Smart Pointers and Transparent Wrappers
impact: MEDIUM
impactDescription: Prevents Deref from becoming a confusing OOP-style inheritance mechanism
tags: [type-safety, deref, smart-pointers, api-design]
---

# Implement Deref Only for Smart Pointers and Transparent Wrappers [MEDIUM]

## Description
`Deref` coercion is what makes `Box<T>`, `Arc<T>`, `String`, and `Vec<T>` ergonomic — the inner type's methods surface through the wrapper transparently. The Rust API Guidelines (C-DEREF) scope this precisely: implement `Deref<Target = T>` only when your type genuinely *is* a smart pointer or a transparent container for `T`. Using it to fake OOP-style inheritance pollutes method resolution — callers can no longer tell, from the call site, whether a method belongs to the wrapper or is silently reached through it — and it makes refactoring hazardous, since adding a method to `T` silently changes what every `Deref`-ing wrapper exposes.

## Bad Example
```rust
struct User {
    name: String,
}

struct AdminUser(User);

// Anti-pattern: using Deref to "inherit" User's fields and methods
impl std::ops::Deref for AdminUser {
    type Target = User;
    fn deref(&self) -> &User {
        &self.0
    }
}

fn greet(admin: &AdminUser) {
    println!("hello, {}", admin.name); // surprising implicit deref
}
```

## Good Example
```rust
struct User {
    name: String,
}

struct AdminUser(User);

impl AdminUser {
    pub fn name(&self) -> &str {
        &self.0.name
    }

    pub fn can_delete_users(&self) -> bool {
        true
    }
}

fn greet(admin: &AdminUser) {
    println!("hello, {}", admin.name()); // explicit, readable
}
```

## Notes
- Legitimate uses: `Box<T>`/`Rc<T>`/`Arc<T>` (pointer indirection), `String` → `str` and `Vec<T>` → `[T]` (owned-to-borrowed containers), `MutexGuard<T>` → `T` (RAII guards granting temporary access).
- A newtype whose entire semantic purpose is "a `T` with extra invariants, otherwise identical" is also a reasonable `Deref` target.
- For everything else — composition, "is-a" relationships you want to expose selectively — write explicit forwarding methods instead.
- `DerefMut` carries the same guidance and the same risk, compounded by the fact that it grants mutable access through the coercion.

## References
- [api-newtype-safety](api-newtype-safety.md)
- [type-newtype-ids](type-newtype-ids.md)
- [own-borrow-over-clone](own-borrow-over-clone.md)
