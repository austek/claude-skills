---
title: Use Newtypes to Enforce Validation at Construction
impact: HIGH
impactDescription: Collapses scattered re-validation into a single guaranteed-valid type
tags: [type-safety, newtype, validation, parse-dont-validate]
---

# Use Newtypes to Enforce Validation at Construction [HIGH]

## Description
When validation lives in a plain `&str` or `String` parameter, every function that receives one has to re-check it, and it's easy to forget one call site. A validated newtype flips this: `Email::new` is the one place invalidity can be rejected, and once you hold an `Email` value, its validity is a property of the type, not something the next function has to re-verify. This is the "parse, don't validate" pattern — it catches errors at the boundary where untrusted data enters the program and makes the invalid state simply unconstructable everywhere past that point.

## Bad Example
```rust
// Validation scattered across every call site that touches a raw string.
fn send_email(to: &str, body: &str) -> Result<(), Error> {
    if !is_valid_email(to) {
        return Err(Error::InvalidEmail);
    }
    todo!()
}

fn add_recipient(list: &mut Vec<String>, email: &str) -> Result<(), Error> {
    if !is_valid_email(email) { // checked again, easy to forget
        return Err(Error::InvalidEmail);
    }
    list.push(email.to_string());
    Ok(())
}
```

## Good Example
```rust
#[derive(Debug, Clone, PartialEq, Eq, Hash)]
pub struct Email(String);

impl Email {
    pub fn new(s: &str) -> Result<Self, EmailError> {
        if is_valid_email(s) {
            Ok(Email(s.to_string()))
        } else {
            Err(EmailError::Invalid(s.to_string()))
        }
    }
}

// No validation needed anywhere past construction -- Email is always valid.
fn add_recipient(list: &mut Vec<Email>, email: Email) {
    list.push(email);
}
```

## Notes
- Common candidates: `Email`, `Url`, `NonEmptyString`, bounded numeric types like `Percentage`, anything with a domain-level "always true" invariant.
- Implement `Deserialize` by hand (deserialize the raw type, then call the validating constructor and map the error) so JSON input is validated automatically on the way in.
- A fallible constructor returning `Result`/`Option` beats a constructor that panics — validation failure at a program boundary is an expected, not exceptional, outcome.
- For values known at compile time, a `const fn` validator plus a macro can push the check to compile time instead of runtime.

## References
- [api-parse-dont-validate](api-parse-dont-validate.md)
- [api-newtype-safety](api-newtype-safety.md)
- [type-newtype-ids](type-newtype-ids.md)
- [conv-fromstr-parsing](conv-fromstr-parsing.md)
