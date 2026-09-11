---
title: Choose Checked vs. Unchecked Exceptions Deliberately
impact: HIGH
impactDescription: keeps exception types communicating recoverability instead of noise
tags: [error-handling, exceptions, api-design]
---

# Choose Checked vs. Unchecked Exceptions Deliberately [HIGH]

## Description
A checked exception (`extends Exception`) forces every caller to either handle it or declare it, which is appropriate exactly when the caller has a realistic, expected recovery path — a missing file, a failed network call, a business rule violation the caller can act on. An unchecked exception (`extends RuntimeException`) is appropriate for programming errors and violated preconditions — a bad argument, a broken invariant, a bug — where the only sane response up the stack is to let it propagate and fail loudly, not to catch and paper over it at every call site. Defaulting everything to unchecked because checked exceptions are inconvenient to declare throws away the compiler's ability to force handling of genuinely recoverable failures; defaulting everything to checked forces callers to handle-or-declare failures they have no meaningful way to recover from, littering signatures with `throws` clauses nobody acts on. Where the failure is recoverable and the codebase already uses Vavr, prefer `Either<Error, T>` over a checked exception entirely — see `error-handling-either-vavr`.

## Bad Example
```java
// Checked exception for a pure programming error — caller cannot meaningfully recover
public class ConfigLoader {
    public Config load(String path) throws IllegalConfigStateException {
        if (path == null) {
            throw new IllegalConfigStateException("path is null"); // this is a bug, not a recoverable case
        }
        // ...
    }
}

// Unchecked exception for a routinely expected, recoverable failure
public class PaymentGateway {
    public void charge(Card card, BigDecimal amount) {
        if (!gatewayIsReachable()) {
            throw new RuntimeException("gateway unreachable"); // caller has a real retry/fallback path, but isn't told to plan for it
        }
    }
}
```

## Good Example
```java
// Unchecked: a null path is a programming error, not something the caller recovers from
public class ConfigLoader {
    public Config load(String path) {
        Objects.requireNonNull(path, "path");
        // ...
    }
}

// Checked: gateway unavailability is expected and recoverable — caller must decide how
public class PaymentGateway {
    public void charge(Card card, BigDecimal amount) throws GatewayUnavailableException {
        if (!gatewayIsReachable()) {
            throw new GatewayUnavailableException("gateway unreachable, retry later");
        }
    }
}

// Caller is compiler-forced to decide: retry, fall back, or propagate
try {
    gateway.charge(card, amount);
} catch (GatewayUnavailableException e) {
    retryQueue.enqueue(new ChargeRequest(card, amount));
}
```

## Notes
- Ask "can a reasonable caller do something useful with this besides logging and crashing?" — yes points to checked (or `Either`), no points to unchecked.
- Never use a checked exception to control ordinary, expected branching (e.g., "not found" during a normal lookup) — that belongs in a return type (`Optional`, `Either`), not the exception hierarchy.
- A library's public API should minimize checked exception types exposed across the boundary — see `error-handling-exception-translation-at-boundary`.

## References
- [Effective Java, 3rd Edition — Item 71: Avoid unnecessary use of checked exceptions](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
- [The Java Tutorials — Unchecked Exceptions: The Controversy](https://docs.oracle.com/javase/tutorial/essential/exceptions/runtime.html)
