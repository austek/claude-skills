---
title: Implement TryFrom for Fallible Conversions
impact: MEDIUM
impactDescription: Unlocks .try_into()? and a free TryInto instead of a one-off function
tags: [conversions, try-from, fallible, api-design]
---

# Implement TryFrom for Fallible Conversions [MEDIUM]

## Description
`TryFrom`/`TryInto` is the standard trait pair for a conversion that can fail. Implementing `TryFrom<T>` gives callers `TryInto` automatically via the standard library's blanket impl — mirroring the `From`/`Into` relationship — which means `.try_into()?` works at every call site without any extra code. It also integrates with generic bounds that constrain `T: TryFrom<U>` and with the wider ecosystem's expectations. A bespoke `fn port_from_u32(n: u32) -> Result<Port, String>` gives none of that: callers must know its exact name, and generic code that wants "a fallible conversion from `u32`" simply can't see it.

## Bad Example
```rust
struct Port(u16);

// Bespoke function -- no .try_into() support, invisible to generic code.
fn port_from_u32(n: u32) -> Result<Port, String> {
    if n > u16::MAX as u32 {
        return Err(format!("port {n} out of range"));
    }
    Ok(Port(n as u16))
}
```

## Good Example
```rust
#[derive(Debug)]
struct Port(u16);

#[derive(Debug)]
struct PortError(u32);

impl TryFrom<u32> for Port {
    type Error = PortError;

    fn try_from(value: u32) -> Result<Self, Self::Error> {
        u16::try_from(value).map(Port).map_err(|_| PortError(value))
    }
}

fn accept_port(n: u32) -> Result<Port, PortError> {
    n.try_into() // standard idiom, works anywhere TryFrom<u32> is implemented
}
```

## Notes
- Implement `TryFrom`, not `TryInto` — the blanket impl in the standard library provides `TryInto` automatically once `TryFrom` exists.
- Use a concrete error type, not `String` or `Box<dyn Error>`, so callers can match on the specific failure reason.
- When the conversion turns out to always succeed, implement `From` instead — `TryFrom` signals fallibility that shouldn't exist if it can't actually happen.

## References
- [api-from-not-into](api-from-not-into.md)
- [conv-fromstr-parsing](conv-fromstr-parsing.md)
- [api-parse-dont-validate](api-parse-dont-validate.md)
