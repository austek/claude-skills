---
title: Name Iterator Types After the Method That Produces Them
impact: LOW
impactDescription: Makes an iterator's origin obvious from its type name alone
tags: [naming, api-design, iterators, types]
---

# Name Iterator Types After the Method That Produces Them [LOW]

## Description
Every iterator-returning method needs a concrete type to name in its signature — even behind `impl Iterator` in the public API, something has to exist underneath — and the standard library's convention is to derive that type's name straight from the method: `iter()` returns `Iter`, `into_iter()` returns `IntoIter`, `keys()` returns `Keys`. Keeping to this mapping means a reader who sees `pub struct Drain<'a, K, V>` in a type's exports already knows, before reading any documentation, that it comes back from a `drain()` call and what shape of items to expect.

## Bad Example
```rust
pub struct MyMapKeyIterator<'a, K> {
    inner: std::slice::Iter<'a, K>,
}

impl<K> MyMap<K> {
    // Type name doesn't echo the method name.
    pub fn keys(&self) -> MyMapKeyIterator<'_, K> { ... }
}
```

## Good Example
```rust
pub struct Keys<'a, K> {
    inner: std::slice::Iter<'a, K>,
}

impl<K> MyMap<K> {
    pub fn keys(&self) -> Keys<'_, K> { ... }
}

impl<'a, K> Iterator for Keys<'a, K> {
    type Item = &'a K;
    fn next(&mut self) -> Option<Self::Item> {
        self.inner.next()
    }
}
```

## Notes
- The mapping generalizes past the three core methods: `chunks()` returns `Chunks`, `windows()` returns `Windows`, and a domain method like `edges()` on a graph type should return `Edges`.
- Naming the type plain `Iterator` is a trap — it shadows `std::iter::Iterator` inside the module and forces every use site to disambiguate.
- Keeping the iterator type in the same module as the method that produces it (rather than off in a `types` module) keeps the name-to-producer link visible in the source layout, not just in the name itself.

## References
- [name-iter-convention](name-iter-convention.md)
- [name-iter-method](name-iter-method.md)
