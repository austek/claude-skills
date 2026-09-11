---
title: Prefix Boolean Methods with is_/has_/can_
impact: MEDIUM
impactDescription: Makes call sites read as the question they answer, no docs needed
tags: [naming, api-design, booleans, readability]
---

# Prefix Boolean Methods with is_/has_/can_ [MEDIUM]

## Description
A method returning `bool` is answering a yes/no question, and the name should state that question rather than just naming a property. `user.active()` is genuinely ambiguous out of context — is it a getter reporting current state, or an action that activates the user? `user.is_active()` cannot be misread that way. The prefix chosen also communicates *what kind* of question is being asked: `is_` for a state check, `has_` for containment or possession, `can_` for a capability or permission check — each one lets `if condition.method()` read like the English sentence it's meant to express.

## Bad Example
```rust
impl Session {
    // Getter or action? The name alone doesn't say.
    fn expired(&self) -> bool {
        self.expires_at < Instant::now()
    }

    fn permission(&self, scope: Scope) -> bool { ... }
}

if session.expired() { ... } // reads ambiguously
```

## Good Example
```rust
impl Session {
    fn is_expired(&self) -> bool {
        self.expires_at < Instant::now()
    }

    fn has_permission(&self, scope: Scope) -> bool { ... }
}

if session.is_expired() && !session.has_permission(Scope::Admin) {
    // reads as plain English
}
```

## Notes
- Prefer the positive form and let the caller negate it (`!is_active()`) rather than adding a mirrored `is_inactive()` — the one exception is `is_empty`, where the check itself is the common case.
- `should_`, `needs_`, and `will_` extend the same idea for recommendation, requirement, and future-action questions respectively (`should_retry`, `needs_auth`).
- Struct *fields* can skip the prefix (`enabled: bool`) since a field access is unambiguously a read — the prefix earns its place specifically on a *method*, where a bare noun could be misread as an action.

## References
- [name-no-get-prefix](name-no-get-prefix.md)
- [name-funcs-snake](name-funcs-snake.md)
