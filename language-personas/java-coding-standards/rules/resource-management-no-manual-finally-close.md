---
title: Never Hand-Write a finally Block to Close a Resource
impact: HIGH
impactDescription: removes the recurring bug class of a swallowed original exception or a null-check omission
tags: [resource-management, try-finally, exception-handling]
---

# Never Hand-Write a finally Block to Close a Resource [HIGH]

## Description
A manual `try`/`finally` that closes a resource is strictly worse than `try-with-resources` at the one thing it exists to do, in two specific ways: if `close()` inside `finally` throws, that exception replaces and hides whatever exception the `try` block was already propagating, destroying the original failure's information; and the `finally` block itself must null-check the resource (it may not have been successfully opened if an earlier line in the `try` threw first) or risk a `NullPointerException` that again masks the real error. `try-with-resources` (Java 7+) gets both of these right automatically — a `close()` failure is attached as a suppressed exception rather than replacing the primary one, and the compiler-generated close logic only closes what was actually assigned. There is no scenario where hand-writing this logic is more correct than the language construct built for it.

## Bad Example
```java
public String readFirstLine(Path path) throws IOException {
    BufferedReader reader = null;
    try {
        reader = Files.newBufferedReader(path);
        return reader.readLine();
    } finally {
        if (reader != null) { // easy to omit, and a real NPE risk if omitted
            reader.close();   // if this throws, it silently replaces any exception from readLine()
        }
    }
}
```

## Good Example
```java
public String readFirstLine(Path path) throws IOException {
    try (BufferedReader reader = Files.newBufferedReader(path)) {
        return reader.readLine();
    } // close() failure here is attached as a suppressed exception, not a replacement
}
```

## Notes
- If a resource genuinely does not implement `AutoCloseable` and cannot be made to (a third-party type outside your control), that is the one legitimate case for a manual `try`/`finally` — wrap it in a small `AutoCloseable` adapter instead if it will be used more than once.
- `Throwable.getSuppressed()` retrieves the suppressed exceptions try-with-resources attaches — worth knowing when reading a stack trace that includes them, since the suppressed exception is often the more informative one.
- This rule is the direct consequence of `resource-management-try-with-resources` and `resource-management-autocloseable-implementation` — implement `AutoCloseable` correctly, then never write the manual equivalent of what the language already does for you.

## References
- [Effective Java, 3rd Edition — Item 9: Prefer try-with-resources to try-finally](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
- [JLS §14.20.3: try-with-resources](https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html#jls-14.20.3)
