---
title: Use into_ for Conversions That Consume self
impact: MEDIUM
impactDescription: Signals a move at the call site, preventing use-after-move confusion
tags: [naming, api-design, ownership, conversions]
---

# Use into_ for Conversions That Consume self [MEDIUM]

## Description
A method that takes `self` by value ends the caller's access to the original value — the compiler will reject any further use of it. The `into_` prefix exists to put that fact directly in the method name, so a reader doesn't have to check the signature to know `wrapper.into_inner()` moves `wrapper` away, unlike `wrapper.inner()` which would read as a harmless borrow. Naming this kind of method `get_x` or `as_x` actively misleads: `get_` suggests a borrow returning a reference, and `as_` is reserved for free reference conversions, neither of which describes a self-consuming transfer.

## Bad Example
```rust
struct TempFile {
    path: PathBuf,
}

impl TempFile {
    // Misnamed: reads like a borrow, but self is consumed here.
    fn get_path(self) -> PathBuf {
        self.path
    }
}
```

## Good Example
```rust
struct TempFile {
    path: PathBuf,
}

impl TempFile {
    // into_ makes the move explicit before the caller even reads the body.
    fn into_path(self) -> PathBuf {
        self.path
    }
}

let temp = TempFile::new();
let path = temp.into_path(); // temp is gone from here on
```

## Notes
- `IntoIterator::into_iter` is the canonical example — it consumes the collection and hands back owned items, in contrast with `iter()` which only borrows.
- The transfer is usually cheap (a field move, no allocation), but "cheap" isn't the point of the name — "ownership moves" is; don't rename a genuinely expensive consuming conversion to `into_` just because it takes `self` by value.
- When a type can be split into several owned parts at once, `into_parts(self) -> (A, B, C)` follows the same convention and reads naturally at the call site.

## References
- [name-as-free](name-as-free.md)
- [name-to-expensive](name-to-expensive.md)
