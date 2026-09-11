---
title: Report Proc-Macro Errors as Spanned Compile Errors, Never by Panicking
impact: HIGH
impactDescription: Turns an opaque "proc macro panicked" into a diagnostic pointing at the offending code
tags: [macros, proc-macros, error-messages]
---

# Report Proc-Macro Errors as Spanned Compile Errors, Never by Panicking [HIGH]

## Description
Because a proc-macro runs as part of the compiler's own process during compilation, a panic inside it — from an explicit `panic!`, or from `.unwrap()`/`.expect()` on something that turned out to be absent — doesn't produce a normal Rust error. It surfaces as a generic "proc macro panicked" message that says nothing about which line in the *user's* source code actually triggered the failure, leaving them to guess based on whatever code they just wrote or changed. The fix is to never let the macro panic at all: convert every failure into a `syn::Error` and return it as a compile-error token stream instead, which the compiler then renders exactly like one of its own diagnostics, complete with a caret pointing at the specific span responsible.

## Bad Example
```rust
use proc_macro::TokenStream;
use syn::{parse_macro_input, Data, DeriveInput};

#[proc_macro_derive(MyTrait)]
pub fn derive_my_trait(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);

    let fields = match input.data {
        Data::Struct(ref s) => &s.fields,
        _ => panic!("MyTrait can only be derived on structs"), // no source location for the user
    };

    let first = fields.iter().next().unwrap(); // "called `unwrap()` on a `None` value"
    let name = first.ident.as_ref().unwrap();

    quote::quote! { impl MyTrait for #name {} }.into()
}
```

## Good Example
```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, Data, DeriveInput, Error};

#[proc_macro_derive(MyTrait)]
pub fn derive_my_trait(input: TokenStream) -> TokenStream {
    derive_my_trait_inner(input).unwrap_or_else(|e| e.to_compile_error().into())
}

fn derive_my_trait_inner(input: TokenStream) -> Result<TokenStream, Error> {
    let input = parse_macro_input!(input as DeriveInput);

    let fields = match &input.data {
        Data::Struct(s) => &s.fields,
        _ => {
            return Err(Error::new_spanned(
                &input.ident,
                "MyTrait can only be derived on structs",
            ));
        }
    };

    let first = fields.iter().next().ok_or_else(|| {
        Error::new_spanned(&input.ident, "MyTrait requires at least one field")
    })?;
    let field_name = first.ident.as_ref().ok_or_else(|| {
        Error::new_spanned(first, "MyTrait requires named fields")
    })?;

    let struct_name = &input.ident;
    Ok(quote! {
        impl MyTrait for #struct_name {
            fn first_field_name() -> &'static str { stringify!(#field_name) }
        }
    }
    .into())
}
```

## Notes
- The common structure is to keep the actual `#[proc_macro_derive]` entry point tiny, delegating to an inner helper typed `fn(...) -> Result<TokenStream2, syn::Error>`, and convert its `Err` case at the boundary with `.unwrap_or_else(|e| e.to_compile_error().into())` — that's the one place a panic-based fallback would otherwise sneak back in.
- `Error::new_spanned(tokens, "message")` attaches a diagnostic to whatever AST node the tokens came from, which is usually preferable to `Error::new(span, "message")`, reserved for cases where only a bare `Span` is available and no convenient token to anchor to.
- Rather than bailing out on the first problem found, `Error::combine` merges several errors into one so a single compile attempt surfaces every issue at once — sparing the user a slow loop of fix-one-error, recompile, find-the-next-one.
- Keeping proc-macro error text lowercase with no trailing period matches the Rust compiler's own convention, so a macro's diagnostics don't stand out as visually different from the compiler's built-in ones.

## References
- [macro-proc-syn-quote](macro-proc-syn-quote.md)
- [err-thiserror-lib](err-thiserror-lib.md)
