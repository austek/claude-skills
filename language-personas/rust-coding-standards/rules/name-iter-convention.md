---
title: Offer iter/iter_mut/into_iter as a Matched Set
impact: MEDIUM
impactDescription: Unlocks for x in &coll, for x in &mut coll, and for x in coll without surprises
tags: [naming, api-design, iterators, ownership]
---

# Offer iter/iter_mut/into_iter as a Matched Set [MEDIUM]

## Description
Rust's `for` loop desugars to a call on `IntoIterator`, and the three names `iter`, `iter_mut`, `into_iter` are how the standard library — and every idiomatic collection type — expose the three ways of walking a container: by shared reference, by mutable reference, and by value. A container that names these methods something else (`elements()`, `get_items()`, `iterate()`) still compiles, but it breaks the pattern every Rust user has memorized, forcing them to check the docs instead of guessing correctly. Implementing `IntoIterator` for the owned type and for both reference types is also what makes `for x in &collection` and `for x in &mut collection` work at all, not just `for x in collection`.

## Bad Example
```rust
struct Roster<T> {
    members: Vec<T>,
}

impl<T> Roster<T> {
    // Compiles, but doesn't plug into `for` loop sugar the way iter() would.
    fn elements(&self) -> impl Iterator<Item = &T> {
        self.members.iter()
    }
}
```

## Good Example
```rust
struct Roster<T> {
    members: Vec<T>,
}

impl<T> Roster<T> {
    fn iter(&self) -> impl Iterator<Item = &T> {
        self.members.iter()
    }

    fn iter_mut(&mut self) -> impl Iterator<Item = &mut T> {
        self.members.iter_mut()
    }
}

impl<T> IntoIterator for Roster<T> {
    type Item = T;
    type IntoIter = std::vec::IntoIter<T>;

    fn into_iter(self) -> Self::IntoIter {
        self.members.into_iter()
    }
}

// Now all three loop forms work as expected:
for m in &roster { }       // borrows, via iter()
for m in &mut roster { }   // mutably borrows, via iter_mut()
for m in roster { }        // consumes, via into_iter()
```

## Notes
- Implementing `IntoIterator` for `&Roster<T>` and `&mut Roster<T>` (in addition to the owned type) is what makes `for x in &roster` resolve — without it, only `for x in roster` compiles.
- `HashMap` extends the same convention with `keys()` and `values()` for partial views over its entries — a domain type with a similar dual structure can follow suit.
- Naming a method `into_iter` when it does *not* consume `self` is worse than a merely unconventional name — it actively lies about ownership.

## References
- [name-iter-method](name-iter-method.md)
- [name-iter-type-match](name-iter-type-match.md)
