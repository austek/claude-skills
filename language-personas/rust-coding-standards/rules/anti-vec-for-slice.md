---
title: Don't Narrow a Function Parameter to &Vec<T>
impact: LOW
impactDescription: Rejects arrays and slices a caller would otherwise pass directly
tags: [anti-pattern, api-design, slices, code-smell]
---

# Don't Narrow a Function Parameter to &Vec<T> [LOW]

## Description
`&Vec<T>` only accepts a reference to a heap-allocated, growable `Vec` — a caller with a fixed-size array or an already-borrowed slice has to convert into a `Vec` first (often via `.to_vec()`, an allocation) just to satisfy a signature that, inside the function body, never actually calls a `Vec`-specific method like `.push()` or `.capacity()`. `&[T]` accepts everything a `&Vec<T>` would (via `Deref`) plus arrays, slices of slices, and any other slice-producing source, with zero loss of capability for a function that's only ever reading or iterating.

## Bad Example
```rust
fn sum(numbers: &Vec<i32>) -> i32 {
    numbers.iter().sum()
}

let arr = [1, 2, 3, 4, 5];
sum(&arr.to_vec()); // forced to allocate a Vec just to call this
```

## Good Example
```rust
fn sum(numbers: &[i32]) -> i32 {
    numbers.iter().sum()
}

sum(&[1, 2, 3, 4, 5]); // array, no allocation
sum(&existing_vec);    // &Vec<i32> -> &[i32] via Deref, still works
```

## Notes
- `clippy::ptr_arg` flags `&Vec<T>` (and `&String`, `&PathBuf`) the same way it flags the other overly-specific-reference anti-patterns — enabling it catches this mechanically rather than relying on review.
- The exception that actually needs `&Vec<T>` or `&mut Vec<T>` is a function that calls something only `Vec` has — `.capacity()`, `.push()`, `.reserve()` — none of which exist on a plain slice.
- `impl AsRef<[T]>` (or the byte-oriented `AsRef<[u8]>`) generalizes further than `&[T]` when a function needs to accept both owned and borrowed slice-like sources equally well — reach for it in that specific case, not as the default over `&[T]`.

## References
- [anti-string-for-str](anti-string-for-str.md)
- [own-slice-over-vec](own-slice-over-vec.md)
- [api-impl-asref](api-impl-asref.md)
