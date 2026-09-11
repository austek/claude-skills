---
title: Wrap a Foreign Type in a Newtype to Satisfy the Orphan Rule
impact: MEDIUM
impactDescription: Unblocks implementing a foreign trait on a foreign type without forking either
tags: [trait-design, coherence, orphan-rule, newtype]
---

# Wrap a Foreign Type in a Newtype to Satisfy the Orphan Rule [MEDIUM]

## Description
Rust's orphan rule requires that for `impl Trait for Type`, either `Trait` or `Type` must be defined in the current crate — this stops two crates from providing conflicting impls for the same pair, which the compiler would have no way to choose between. Trying to implement a foreign trait (`std::fmt::Display`) on a foreign type (`Vec<i32>`) is rejected outright, even with a type parameter added. The fix is to define a local newtype wrapping the foreign type, then implement the foreign trait on that local wrapper instead — the wrapper being local satisfies the rule.

## Bad Example
```rust
use std::fmt;

// error[E0117]: only traits defined in the current crate can be implemented
// for types defined outside of the crate -- both Display and Vec are foreign.
// impl fmt::Display for Vec<i32> { ... }
```

## Good Example
```rust
use std::fmt;

// Local newtype wrapping the foreign Vec<i32>.
#[repr(transparent)]
struct CommaSeparated(Vec<i32>);

// Display is foreign, but CommaSeparated is local -- orphan rule satisfied.
impl fmt::Display for CommaSeparated {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        let mut iter = self.0.iter().peekable();
        while let Some(n) = iter.next() {
            write!(f, "{n}")?;
            if iter.peek().is_some() {
                write!(f, ", ")?;
            }
        }
        Ok(())
    }
}
```

## Notes
- The orphan rule rejects `impl<T> ForeignTrait for ForeignType<T>` even with a type parameter — wrapping is the only escape, not a generic workaround.
- `#[repr(transparent)]` is mandatory when the wrapper needs the same ABI as the inner type (FFI, `transmute`-based pointer casts); for purely logical wrapping it's optional but still good practice.
- Provide `From`/`Into` conversions and `inner()`/`into_inner()` accessors so callers can move in and out of the wrapper without friction.
- This is also the correct way to add trait impls to a type that comes from a transitive dependency you don't control directly.

## References
- [api-newtype-safety](api-newtype-safety.md)
- [type-repr-transparent](type-repr-transparent.md)
- [trait-blanket-impl](trait-blanket-impl.md)
