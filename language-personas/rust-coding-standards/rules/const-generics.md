---
title: Parameterize Over Sizes with Const Generics
impact: MEDIUM
impactDescription: Eliminates heap allocation and per-size code duplication with zero runtime overhead
tags: [const, generics, monomorphization, zero-cost]
---

# Parameterize Over Sizes with Const Generics [MEDIUM]

## Description
A const generic parameter, written `<const N: usize>`, lets a type or function stay generic over a size or other fixed value the same way ordinary generics stay generic over a type. Before this existed, supporting several array sizes meant either duplicating the function per size, falling back to a runtime-length `Vec`, or reaching for a trait-object workaround — all of which either lose the compile-time size information or add indirection. The compiler instead generates one specialized version of the code per distinct `N` actually used by callers, exactly as it does for a normal generic type parameter, so calling `sum([1,2,3,4])` and `sum([0i32; 8])` produces two independent, fully-inlined functions rather than one function paying a size-agnostic runtime cost. Because the size lives in the type itself, mismatched capacities are caught by the type checker at the call site instead of surfacing as a runtime bounds failure later.

## Bad Example
```rust
// Works only for one fixed size — must be copy-pasted per size
fn sum_4(arr: [i32; 4]) -> i32 {
    arr.iter().sum()
}

fn sum_8(arr: [i32; 8]) -> i32 {
    arr.iter().sum()
}

// Carries a runtime length and heap-allocates — no compile-time capacity bound
struct Buffer {
    data: Vec<u8>,
    capacity: usize,
}
```

## Good Example
```rust
// One generic function works for any array size; N is inferred from the argument
fn sum<const N: usize>(arr: [i32; N]) -> i32 {
    arr.iter().sum()
}

let total = sum([1, 2, 3, 4]);  // N = 4, inferred
let total8 = sum([0i32; 8]);    // N = 8, inferred

// Stack-allocated buffer parameterized by capacity — no heap, no runtime length field
struct Buffer<const N: usize> {
    data: [u8; N],
    len: usize,
}

impl<const N: usize> Buffer<N> {
    const fn new() -> Self {
        Self { data: [0u8; N], len: 0 }
    }

    fn push(&mut self, byte: u8) -> bool {
        if self.len < N {
            self.data[self.len] = byte;
            self.len += 1;
            true
        } else {
            false
        }
    }
}

// Capacity is part of the type — mismatches caught at compile time, not runtime
let mut small: Buffer<8> = Buffer::new();
let mut large: Buffer<1024> = Buffer::new();
```

## Notes
- Default values for a const generic parameter — `struct Buf<const N: usize = 64>` — have been stable since Rust 1.59, well before most of the ecosystem started relying on the feature; a caller can lean on the default and only override it when a non-default size is actually needed.
- The set of legal const generic parameter types is still deliberately narrow on stable Rust: integers, `bool`, and `char` work, but floating-point values and arbitrary custom types don't yet.
- Most call sites never need to name `N` explicitly, since the compiler infers it from whatever array or value is passed in — reserve the turbofish form (`sum::<4>(...)`) for the rarer case where nothing in the call lets the compiler infer it on its own.

## References
- [const-fn](const-fn.md)
- [mem-assert-type-size](mem-assert-type-size.md)
