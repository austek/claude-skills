---
title: Build Traits From a Few Required Methods Plus Defaults
impact: LOW
impactDescription: Shrinks every implementor's burden to the essential logic only
tags: [trait-design, default-methods, api-design]
---

# Build Traits From a Few Required Methods Plus Defaults [LOW]

## Description
A trait with only required methods puts the full implementation burden on every consumer, even for logic that's a mechanical composition of one core method — `std::iter::Iterator` avoids this by building `map`, `filter`, `fold`, and dozens more on top of a single required `next`. Default method bodies let implementors write only the essential logic and get the rest for free, and they double as documentation of the canonical relationship between methods. An implementor can still override a default when it's provably faster for that type, without changing the trait's observable contract.

## Bad Example
```rust
// Every implementor must hand-write all three, though two are mechanical
// compositions of the first -- duplicated logic drifts out of sync.
trait Summarise {
    fn sentences(&self) -> Vec<String>;
    fn first_sentence(&self) -> Option<String>;
    fn word_count(&self) -> usize;
}
```

## Good Example
```rust
trait Summarise {
    // Required: the only thing implementors must provide.
    fn sentences(&self) -> Vec<String>;

    // Defaulted: free for every implementor.
    fn first_sentence(&self) -> Option<String> {
        self.sentences().into_iter().next()
    }

    fn word_count(&self) -> usize {
        self.sentences().join(" ").split_whitespace().count()
    }
}

// Minimal impl -- one method, the rest comes for free.
struct Article { body: String }

impl Summarise for Article {
    fn sentences(&self) -> Vec<String> {
        self.body.split('.').map(str::trim).map(str::to_owned).collect()
    }
}
```

## Notes
- Keep the required set to the minimum orthogonal core — ideally one or two methods everything else can be built from.
- Default implementations should only call other methods on `Self`, never reach into external state the trait doesn't own.
- An override must preserve the default's observable semantics; changing performance characteristics is fine, changing behavior isn't.
- This pattern underlies `Iterator`, `std::io::Read`, and `std::io::Write` — document which methods are required vs. defaulted in the trait's own doc comment.

## References
- [api-extension-trait](api-extension-trait.md)
- [trait-associated-type-vs-generic](trait-associated-type-vs-generic.md)
- [trait-blanket-impl](trait-blanket-impl.md)
