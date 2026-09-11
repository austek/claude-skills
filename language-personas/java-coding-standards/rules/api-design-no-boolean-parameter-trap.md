---
title: Avoid the Boolean Parameter Trap — Name the Choice
impact: MEDIUM
impactDescription: call sites state what a flag means instead of leaving readers to count positional booleans
tags: [api-design, readability, method-signatures]
---

# Avoid the Boolean Parameter Trap — Name the Choice [MEDIUM]

## Description
A `boolean` parameter carries no meaning at the call site — `createUser(name, true)` tells a reader nothing about what `true` turns on without checking the method signature, and a second or third boolean parameter compounds the problem into an unreadable, easily transposed sequence of `true`/`false` literals. The parameter is also a sign the method is silently doing two different things selected by a flag, which is usually better expressed as two differently named methods or as an enum whose constants name each option. Both fixes make the call site self-explanatory without needing to open the method's declaration.

## Bad Example
```java
public User createUser(String name, boolean sendWelcomeEmail, boolean isAdmin) { /* ... */ }

// What do these mean? Reader must open the method signature to find out.
createUser("Alice", true, false);
```

## Good Example
```java
// Option A: split into differently named methods
public User createUser(String name) { /* ... */ }
public User createUserAndWelcome(String name) { /* ... */ }

// Option B: name the choice with an enum where more than one flag is involved
public enum Role { STANDARD, ADMIN }

public User createUser(String name, Role role, EmailPreference emailPreference) { /* ... */ }

createUser("Alice", Role.STANDARD, EmailPreference.SEND_WELCOME);
```

## Notes
- A single, unambiguous boolean where the parameter name already makes the meaning obvious at the call site (`repository.findAll(includeArchived)` with a named variable, not a bare literal) is a milder case — the real problem is a bare `true`/`false` literal at the call site with no name attached.
- An enum scales better than a boolean the moment a third option becomes plausible — a boolean can never grow past two states without becoming a breaking signature change.
- This does not apply to record/constructor components read back via a named accessor (`request.sendWelcomeEmail()`) — the trap is specifically about unnamed literals at a call site.

## References
- [Effective Java, 3rd Edition — Item 51: Design method signatures carefully](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
