---
title: Keep Iterator-Producing Methods to a Fixed Naming Vocabulary
impact: MEDIUM
impactDescription: Lets callers guess the right method name without checking docs
tags: [naming, api-design, iterators, discoverability]
---

# Keep Iterator-Producing Methods to a Fixed Naming Vocabulary [MEDIUM]

## Description
Once `iter`, `iter_mut`, and `into_iter` are established as the names for "give me an iterator over this," every other method on the same type that returns an iterator should follow the same small vocabulary rather than inventing new verbs. `HashMap::keys()`, `HashMap::values()`, `Vec::drain()`, `[T]::chunks()` all sit inside this vocabulary — a reader who's used one collection can predict a method's name on an unfamiliar one without opening documentation. A type that names its equivalent methods `get_all_keys()` or `iterate_values()` breaks that transferable intuition even though nothing is functionally wrong with it.

## Bad Example
```rust
impl<K, V> IndexMap<K, V> {
    // Works, but doesn't match the vocabulary readers already know.
    fn get_iterator(&self) -> impl Iterator<Item = (&K, &V)> { ... }
    fn to_iter(self) -> impl Iterator<Item = (K, V)> { ... }
}
```

## Good Example
```rust
impl<K, V> IndexMap<K, V> {
    fn iter(&self) -> impl Iterator<Item = (&K, &V)> { ... }
    fn iter_mut(&mut self) -> impl Iterator<Item = (&K, &mut V)> { ... }
    fn keys(&self) -> impl Iterator<Item = &K> { ... }
    fn values(&self) -> impl Iterator<Item = &V> { ... }
    fn drain(&mut self) -> impl Iterator<Item = (K, V)> { ... }
}

impl<K, V> IntoIterator for IndexMap<K, V> {
    type Item = (K, V);
    type IntoIter = std::vec::IntoIter<(K, V)>;

    fn into_iter(self) -> Self::IntoIter { ... }
}
```

## Notes
- `drain()` is the "iterate and empty the container" verb — it borrows mutably, yields owned items, and leaves the collection empty afterward, distinct from `into_iter()` which consumes the collection itself.
- A domain-specific iterator method (`graph.neighbors(node)`) doesn't need to match this vocabulary exactly — the fixed names apply to the generic "walk the whole thing" methods, not to every iterator-returning method a type might expose.
- Document what each borrowing variant yields (`&T`, `&mut T`, or `T`) directly on the method, since the name alone communicates *which* variant but not the concrete item type.

## References
- [name-iter-convention](name-iter-convention.md)
- [name-iter-type-match](name-iter-type-match.md)
