---
title: Build Procedural Macros with syn, quote, and proc-macro2
impact: MEDIUM
impactDescription: Replaces brittle manual token parsing with a typed, unit-testable AST
tags: [macros, proc-macros, tooling]
---

# Build Procedural Macros with syn, quote, and proc-macro2 [MEDIUM]

## Description
`proc_macro::TokenStream` is a bare, untyped stream of tokens, and walking it by hand to find, say, "the struct's name" means writing code that assumes a specific shape for the input — and that assumption breaks the moment someone passes a struct with generics, an attribute, or any syntax the manual walk didn't anticipate. Three crates together turn that fragile approach into something closer to normal Rust development: `syn` parses the token stream into a proper typed AST that already understands generics, attributes, and the rest of Rust's grammar; `quote` generates new code from that AST using a template-like syntax instead of manual string concatenation; and `proc-macro2` provides span-aware token types that work identically whether they're running inside a real compilation or inside a plain `#[test]` function. That last point matters more than it might seem — it means the code-generation logic itself can be exercised by ordinary unit tests, without needing to actually compile a separate crate that invokes the macro.

## Bad Example
```rust
// Manually iterating tokens to find a struct name — brittle and breaks on generics
use proc_macro::TokenStream;

#[proc_macro_derive(Hello)]
pub fn derive_hello(input: TokenStream) -> TokenStream {
    let mut iter = input.into_iter();
    iter.next(); // skip "struct"
    let name = iter.next().unwrap().to_string();
    format!("impl Hello for {name} {{ fn hello(&self) {{ println!(\"hello\"); }} }}")
        .parse()
        .unwrap()
}
```

## Good Example
```toml
# Cargo.toml for the derive crate
[dependencies]
syn = { version = "2", features = ["derive"] }
quote = "1"
proc-macro2 = "1"
```
```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, DeriveInput};

#[proc_macro_derive(Hello)]
pub fn derive_hello(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    let name = &input.ident;
    let (impl_generics, ty_generics, where_clause) = input.generics.split_for_impl();

    // quote! quasi-quotes Rust tokens; #name splices the identifier in
    let expanded = quote! {
        impl #impl_generics Hello for #name #ty_generics #where_clause {
            fn hello(&self) {
                println!("hello from {}", stringify!(#name));
            }
        }
    };
    expanded.into()
}
```

## Notes
- `syn`'s feature flags directly trade compile time for parsing coverage — the `full` feature understands every Rust syntax construct but noticeably lengthens the derive crate's own build, while `derive` covers the narrower grammar that shows up in most `#[proc_macro_derive]` inputs and compiles faster.
- `quote_spanned! { span => ... }` is the variant of `quote!` that lets generated code inherit a specific span, usually taken from a particular field or attribute in the input — doing this means a compiler error about the *generated* code still points at a meaningful location in the *user's* source, rather than nowhere in particular.
- Since a `proc-macro2::TokenStream` behaves the same inside a `#[test]` as inside a real macro invocation, the generation logic is worth factoring into a plain function that a unit test can call directly, using `syn::parse_quote!` to build sample inputs — this sidesteps needing an actual second crate just to exercise the macro end-to-end.

## References
- [macro-proc-error-spans](macro-proc-error-spans.md)
- [macro-proc-two-crate](macro-proc-two-crate.md)
