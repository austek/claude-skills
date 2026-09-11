---
title: Use Enums for Mutually Exclusive States
impact: HIGH
impactDescription: Makes invalid state combinations unrepresentable instead of merely unlikely
tags: [type-safety, enums, state-machines, invariants]
---

# Use Enums for Mutually Exclusive States [HIGH]

## Description
When a value can be in exactly one of several states, a struct built from independent boolean flags and optional fields can represent combinations that should never occur — connected and disconnected at once, authenticated but with no socket. An enum collapses those possibilities to exactly the valid ones: each variant carries only the data that state actually has, and the compiler forces every match to handle every variant. This turns a class of "impossible but representable" bugs into a compile-time guarantee instead of a runtime invariant someone has to remember to check.

## Bad Example
```rust
struct Connection {
    is_connected: bool,
    is_authenticated: bool,
    is_disconnected: bool, // can all three be true? false?
    socket: Option<TcpStream>,
    credentials: Option<Credentials>,
}
// is_connected && is_disconnected is a contradiction the type allows.
```

## Good Example
```rust
enum ConnectionState {
    Disconnected,
    Connecting { address: SocketAddr },
    Connected { socket: TcpStream },
    Authenticated { socket: TcpStream, session: Session },
    Failed { error: ConnectionError },
}

struct Connection {
    state: ConnectionState,
}

fn handle_connection(conn: &Connection) {
    match &conn.state {
        ConnectionState::Disconnected => println!("not connected"),
        ConnectionState::Connecting { address } => println!("connecting to {address}"),
        ConnectionState::Connected { .. } => println!("connected, not authenticated"),
        ConnectionState::Authenticated { session, .. } => println!("authenticated as {}", session.user),
        ConnectionState::Failed { error } => println!("failed: {error}"),
    } // compiler error if any variant is missing
}
```

## Notes
- `Option<T>` and `Result<T, E>` are themselves state enums for "might not exist" and "might have failed" — prefer them over sentinel values for the same reason.
- State transitions read cleanly with `std::mem::replace` to take ownership of the old state while computing the new one, especially when a variant owns non-`Copy` data like a `TcpStream`.
- A struct of independent `bool` flags is the strongest smell that an enum belongs here — especially once a comment has to explain which combinations are actually valid.
- `#[non_exhaustive]` on a state enum keeps it forward-compatible across crate versions without forcing every external match to add a wildcard arm prematurely.

## References
- [api-typestate](api-typestate.md)
- [api-non-exhaustive](api-non-exhaustive.md)
- [type-option-nullable](type-option-nullable.md)
