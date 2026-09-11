---
title: Never Catch Throwable or Error
impact: CRITICAL
impactDescription: stops OutOfMemoryError and other unrecoverable JVM failures from being swallowed and the process left in a corrupt state
tags: [anti-patterns, error-handling, exceptions]
---

# Never Catch Throwable or Error [CRITICAL]

## Description
`Throwable` is the root of both `Exception` (conditions a well-written application might reasonably catch and handle) and `Error` (conditions indicating a serious problem — `OutOfMemoryError`, `StackOverflowError`, `NoClassDefFoundError` — that a normal application has no reliable way to recover from). Catching `Throwable` or `Error` directly, especially with a broad `catch` that logs and continues, means the process keeps running after the JVM itself has signaled it's in a state it cannot guarantee: memory exhausted, a thread's stack blown, a class failed to load. Continuing past that point risks silent data corruption, partially completed transactions, or a slow cascade of further failures that are far harder to diagnose than a clean crash would have been — masking exactly the failure that most needs to surface immediately, to logs, to an operator, and typically to a process restart.

Catch specific, expected exception types instead — the checked or unchecked exceptions a method's own contract documents (see `javadoc-throws-documentation`) — and let unanticipated `Error`s propagate so the JVM, a supervisor process, or a container orchestrator can respond to them appropriately.

## Bad Example
```java
public void processMessage(Message message) {
    try {
        handler.handle(message);
    } catch (Throwable t) { // swallows OutOfMemoryError, StackOverflowError — the JVM is now in an unknown state
        log.error("Failed to process message", t);
    }
}
```

## Good Example
```java
public void processMessage(Message message) {
    try {
        handler.handle(message);
    } catch (MessageProcessingException e) { // a specific, documented, recoverable failure
        log.error("Failed to process message {}", message.id(), e);
        deadLetterQueue.send(message);
    }
    // an Error propagates uncaught, as it should — the JVM/orchestrator handles process-level failure
}
```

## Notes
- A top-level "don't crash the whole server on one bad request" boundary (a servlet filter, a message-consumer loop) may reasonably catch `RuntimeException`, but even there `Error` should not be caught — most application frameworks' own top-level handlers are careful to exclude it.
- `AssertionError` is technically an `Error` but is sometimes caught deliberately in test infrastructure — that's a narrow, test-specific exception to this rule, not a justification for catching `Error` in application code.
- See `error-handling-no-swallowed-exceptions` for the related, narrower problem of catching a specific exception and discarding it without logging or rethrowing.

## References
- [java.lang.Throwable specification](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Throwable.html)
- [Effective Java, 3rd Edition — Item 77: Don't ignore exceptions](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
