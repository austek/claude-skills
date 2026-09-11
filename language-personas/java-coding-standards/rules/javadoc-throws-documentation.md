---
title: Document Every Checked and Meaningful Unchecked Throws
impact: HIGH
impactDescription: turns an exception a caller must catch or expect from a runtime surprise into a documented contract
tags: [javadoc, error-handling, documentation]
---

# Document Every Checked and Meaningful Unchecked Throws [HIGH]

## Description
Every checked exception a method declares belongs in an `@throws` tag explaining the condition that triggers it — the compiler already forces callers to handle a checked exception, so the only missing piece is *when* it happens, which only the doc comment can supply. Unchecked exceptions are not compiler-enforced, but a method that deliberately throws one as part of its contract (`IllegalArgumentException` for an invalid argument, `IllegalStateException` for a misused API) should still document it — a caller has no way to discover an unchecked contract short of reading the implementation or hitting it in production. Accidental unchecked exceptions that indicate a bug rather than an intended contract (a `NullPointerException` from a missed null check) do not need documentation — documenting them would just paper over a bug that should be fixed instead.

An `@throws` tag is precise about the trigger condition, not just the type: "@throws IllegalArgumentException if `rate` is negative" tells a caller what to avoid; "@throws IllegalArgumentException if the argument is invalid" tells them nothing they couldn't guess from the type name alone.

## Bad Example
```java
/**
 * Parses a configuration file.
 */
public Config parse(Path path) throws IOException {
    if (!Files.exists(path)) {
        throw new NoSuchFileException(path.toString());
    }
    return doParse(path);
}
```

## Good Example
```java
/**
 * Parses the configuration file at {@code path}.
 *
 * @throws NoSuchFileException if {@code path} does not exist
 * @throws IOException if the file exists but cannot be read
 * @throws ConfigFormatException if the file's contents are not valid configuration syntax
 */
public Config parse(Path path) throws IOException {
    if (!Files.exists(path)) {
        throw new NoSuchFileException(path.toString());
    }
    return doParse(path);
}
```

## Notes
- `NoSuchFileException` is a subtype of `IOException` — documenting it separately, as above, tells a caller it can be caught and handled distinctly from a generic read failure.
- For the choice between a checked exception and `Either`, see `error-handling-checked-vs-unchecked` and `error-handling-either-vavr` — this rule applies to whichever mechanism the method actually uses.
- Keep `@throws` conditions in sync with the actual throw sites during a refactor — a stale `@throws` is as misleading as a missing one.

## References
- [How to Write Doc Comments for the Javadoc Tool (Oracle)](https://www.oracle.com/technical-resources/articles/java/javadoc-tool.html)
- [Effective Java, 3rd Edition — Item 74: Document all exceptions thrown by each method](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
