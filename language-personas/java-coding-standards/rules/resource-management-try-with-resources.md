---
title: Always Use try-with-resources for AutoCloseable Values
impact: CRITICAL
impactDescription: closes a resource on every exit path, including exceptions, with no boilerplate to get wrong
tags: [resource-management, try-with-resources, autocloseable]
---

# Always Use try-with-resources for AutoCloseable Values [CRITICAL]

## Description
Any `AutoCloseable` (a `Stream` from `Files.lines`, an `InputStream`, a `Connection`, a `Lock`) must be closed on every path out of the code that opened it, including an exception thrown partway through — a resource closed only after a normal return leaks on the exception path, and file handles, sockets, and connections are finite. `try-with-resources` (Java 7+) closes every resource declared in its parenthesized clause automatically when the block exits, by any path, in reverse declaration order, and does so correctly even when the resource's own `close()` itself throws (that exception is added as a suppressed exception on the primary one, rather than replacing it and hiding the original failure). There is no correct equivalent written by hand with a manual `finally` block that isn't strictly more code for the same guarantee — see `resource-management-no-manual-finally-close`.

## Bad Example
```java
public String readFirstLine(Path path) throws IOException {
    BufferedReader reader = Files.newBufferedReader(path);
    String line = reader.readLine(); // if this throws, reader is never closed
    reader.close();
    return line;
}
```

## Good Example
```java
public String readFirstLine(Path path) throws IOException {
    try (BufferedReader reader = Files.newBufferedReader(path)) {
        return reader.readLine(); // reader.close() runs on every exit path, exception or not
    }
}
```

## Notes
- Multiple resources declared in one `try (...)` close in reverse order of declaration, each independent of whether an earlier one's `close()` threw.
- Java 9+ allows an effectively-final variable declared outside the `try` to be used directly in the resources clause (`try (existingResource) { ... }`) without redeclaring it.
- A resource whose `close()` is a no-op or cheap to call still belongs in try-with-resources — the guarantee, not the cost of closing, is the point.

## References
- [JLS §14.20.3: try-with-resources](https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html#jls-14.20.3)
- [Effective Java, 3rd Edition — Item 9: Prefer try-with-resources to try-finally](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
