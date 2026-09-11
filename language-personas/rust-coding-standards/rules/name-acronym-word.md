---
title: Capitalize Acronyms as Ordinary Words
impact: MEDIUM
impactDescription: Keeps clippy::upper_case_acronyms quiet and identifiers scannable
tags: [naming, style, acronyms, readability, clippy]
---

# Capitalize Acronyms as Ordinary Words [MEDIUM]

## Description
Rust's `UpperCamelCase` convention wants each acronym treated as a single capitalized word, not preserved as an all-caps block. The reason is visual: two adjacent runs of capitals remove the boundary a reader normally uses to split a compound identifier into words. Faced with `HTTPSXMLParser`, a reader has to stop and consciously work out where one acronym ends and the next begins; `HttpsXmlParser` reads left to right without that pause. This isn't only a style preference — `clippy::upper_case_acronyms` fires by default on public items that keep the old casing, so an all-caps acronym shows up as lint noise in CI. The standard library commits to this pattern everywhere: `TcpStream`, `UdpSocket`, `IpAddr`, never `TCPStream` or `IPAddr`.

## Bad Example
```rust
struct HTTPSClient {
    base_url: String,
}

fn parseXMLPayload(raw: &str) -> XMLDocument {
    todo!()
}

// Ambiguous split: is this I/O-Error or something else?
struct IOError;
```

## Good Example
```rust
struct HttpsClient {
    base_url: String,
}

fn parse_xml_payload(raw: &str) -> XmlDocument {
    todo!()
}

// One unambiguous boundary: Io + Error
struct IoError;
```

## Notes
- Two-letter acronyms (`Io`, `Id`) sit in a gray zone — `IoHandler` and `IOHandler` both clear clippy — but treating them as ordinary words keeps the convention uniform across the codebase.
- In `snake_case` positions the acronym collapses to lowercase entirely: `parse_xml`, `fetch_uuid`, never `parse_XML`.
- Reach for `#[allow(clippy::upper_case_acronyms)]` only when an external spec fixes the exact casing (protocol names like `HTTP2`) — treat it as an exception that needs a reason, not a default.

## References
- [name-types-camel](name-types-camel.md)
- [name-funcs-snake](name-funcs-snake.md)
