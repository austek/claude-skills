---
title: Never Write an Empty Catch Block
impact: CRITICAL
impactDescription: prevents silent data loss and failures that surface only much later, far from the cause
tags: [error-handling, exceptions, logging]
---

# Never Write an Empty Catch Block [CRITICAL]

## Description
An empty `catch` block — or one containing only a comment — discards the exception and everything it would have told a future debugger: what failed, why, and where. The code continues as if nothing happened, often leaving the program in an inconsistent state that surfaces as a confusing failure somewhere else entirely, far from the real cause, sometimes only in production under load. Every `catch` block must do at least one of: log the exception with its stack trace and enough context to act on it, rethrow it (wrapped or as-is), or recover with a specific, deliberate fallback that is itself visible in code and, when warranted, logged. If a caught exception truly cannot occur or truly does not matter, that must be a commented, reasoned decision, not silence.

## Bad Example
```java
public void processQueue(List<Message> messages) {
    for (Message message : messages) {
        try {
            handler.process(message);
        } catch (Exception e) {
            // swallowed — no log, no rethrow, message silently disappears
        }
    }
}
```

## Good Example
```java
public void processQueue(List<Message> messages) {
    for (Message message : messages) {
        try {
            handler.process(message);
        } catch (ProcessingException e) {
            log.error("Failed to process message {}: {}", message.id(), e.getMessage(), e);
            deadLetterQueue.send(message);
        }
    }
}

// A caught exception that genuinely cannot occur still gets a stated reason, not silence
try {
    return objectMapper.readValue(STATIC_TEMPLATE_JSON, Config.class);
} catch (JsonProcessingException e) {
    // STATIC_TEMPLATE_JSON is a compile-time constant validated by a unit test — cannot fail at runtime
    throw new IllegalStateException("unreachable: static template JSON is malformed", e);
}
```

## Notes
- `catch (Exception e) { e.printStackTrace(); }` is only marginally better than an empty block: `printStackTrace()` writes to `System.err`, bypasses the logging framework's levels, filtering, and aggregation, and is easy to lose in production log pipelines — use the logger.
- Catching `Exception` or `Throwable` broadly to swallow "whatever might go wrong" hides bugs the same way an empty block does; catch the specific exception type the code can actually handle.
- Static analyzers (SonarQube's `S108`/`S1166`, SpotBugs `REC_CATCH_EXCEPTION`, PMD `EmptyCatchBlock`) exist specifically to catch this pattern — never suppress the warning, fix the block, per the workplace rule against silently disabling lint guards.
- Rethrowing after logging is not double-handling — logging locally captures context the caller may not have (which message, which loop iteration); the rethrow lets the caller still decide the ultimate outcome.

## References
- [SEI CERT Oracle Coding Standard for Java — ERR00-J: Do not suppress or ignore checked exceptions](https://wiki.sei.cmu.edu/confluence/display/java/ERR00-J.+Do+not+suppress+or+ignore+checked+exceptions)
- [SonarSource Rule S108: Nested blocks of code should not be left empty](https://rules.sonarsource.com/java/RSPEC-108/)
