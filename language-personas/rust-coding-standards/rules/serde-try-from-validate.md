---
title: Validate During Deserialization with serde try_from
impact: HIGH
impactDescription: Makes an invalid value unrepresentable instead of catching it after construction
tags: [serde, serialization, validation, parse-dont-validate]
---

# Validate During Deserialization with serde try_from [HIGH]

## Description
The usual pattern of "deserialize, then call a validation function on the result" has a structural weakness: for however brief a moment, an instance of the type exists holding data nobody has checked yet, and nothing in the type system stops a future call site from skipping the validation step entirely. The parse-don't-validate principle argues for closing that gap by making the invalid state impossible to construct rather than merely inconvenient to reach. `#[serde(try_from = "Raw")]` gets there by rerouting deserialization through a `TryFrom` implementation: serde first deserializes the simpler `Raw` type, then hands it to `TryFrom::try_from`, and only a successful conversion ever produces a value of the target type — a value that fails validation simply never comes into existence, rather than existing invalidly until someone happens to check it.

## Bad Example
```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug, Clone)]
struct Email(String);

impl Email {
    fn new(s: String) -> Result<Self, String> {
        if s.contains('@') { Ok(Email(s)) } else { Err(format!("invalid email: {s}")) }
    }
}

// Caller must remember to validate after deserializing — easy to skip
fn process(json: &str) -> Result<(), Box<dyn std::error::Error>> {
    let email: Email = serde_json::from_str(json)?;
    // Nothing stops "notanemail" from being deserialized and used here
    println!("{email:?}");
    Ok(())
}
```

## Good Example
```rust
use serde::{Serialize, Deserialize};

#[derive(Debug, Clone, Serialize, Deserialize)]
#[serde(try_from = "String", into = "String")]
struct ValidatedEmail(String);

impl TryFrom<String> for ValidatedEmail {
    type Error = String;

    fn try_from(s: String) -> Result<Self, Self::Error> {
        if s.contains('@') && !s.starts_with('@') && !s.ends_with('@') {
            Ok(ValidatedEmail(s))
        } else {
            Err(format!("invalid email address: {s}"))
        }
    }
}

impl From<ValidatedEmail> for String {
    fn from(e: ValidatedEmail) -> String {
        e.0
    }
}

fn main() {
    // Invalid email is rejected at parse time — never enters the program
    let bad = serde_json::from_str::<ValidatedEmail>("\"notanemail\"");
    assert!(bad.is_err());
}
```

## Notes
- Mechanically, `try_from = "Raw"` deserializes into `Raw` and then feeds it to `T::try_from`, turning any `Err` the conversion returns into a normal serde deserialization error the caller already knows how to handle. The matching `into = "Raw"` attribute handles the opposite direction by calling `Raw::from(value)`, which requires the target type to also implement `Clone`, since serde may need to clone the value before consuming it in the conversion.
- `Raw` itself needs to implement `Deserialize` for the `try_from` side to work, and the target type needs `From<T> for Raw` (equivalently, `Into<Raw>`) for the `into` side — both are ordinary trait implementations, not serde-specific machinery.
- Because the validation logic lives in a plain `TryFrom` impl rather than inside a serde attribute, the exact same function can validate a CLI argument, a form submission, or any other untrusted input — writing the check once covers every entry point, not only the JSON path.
- Adding `try_from`/`into` replaces serde's normal field-by-field derive entirely for that type, so it can't be layered on top of an otherwise-ordinary `#[derive(Serialize, Deserialize)]` — it's one or the other.

## References
- [api-parse-dont-validate](api-parse-dont-validate.md)
- [type-newtype-validated](type-newtype-validated.md)
- [serde-custom-with](serde-custom-with.md)
