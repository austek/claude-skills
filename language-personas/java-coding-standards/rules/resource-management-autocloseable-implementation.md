---
title: Implement AutoCloseable Correctly for Custom Resource-Holding Classes
impact: HIGH
impactDescription: makes the class usable in try-with-resources and safe to close more than once
tags: [resource-management, autocloseable, api-design]
---

# Implement AutoCloseable Correctly for Custom Resource-Holding Classes [HIGH]

## Description
Any class that owns an external resource — a file handle, a socket, a native handle, a pooled connection — should implement `AutoCloseable` so it can be used with `try-with-resources` (`resource-management-try-with-resources`) rather than requiring callers to remember a manual cleanup call. A correct implementation makes `close()` idempotent (safe to call more than once, since a caller closing explicitly and then again via try-with-resources, or closing from two code paths under error handling, is a realistic scenario) and closes every resource the class owns, not just the first one, even if closing an earlier one throws. Prefer overriding `close()` to declare no checked exception (or a specific one) rather than the broad `Exception` the interface method permits — a narrower signature is easier for callers to handle without a catch-all.

## Bad Example
```java
public class FileArchiver implements AutoCloseable {
    private final InputStream input;
    private final OutputStream output;
    private boolean closed = false;

    public void close() throws Exception {
        input.close();  // if this throws, output.close() never runs — leaks the output handle
        output.close();
        closed = true;  // never reached on the exception path, so a second close() re-attempts input.close()
    }
}
```

## Good Example
```java
public class FileArchiver implements AutoCloseable {
    private final InputStream input;
    private final OutputStream output;
    private boolean closed = false;

    @Override
    public void close() throws IOException {
        if (closed) {
            return; // idempotent: a second close() is a no-op, not an error
        }
        closed = true;
        try {
            input.close();
        } finally {
            output.close(); // always attempted, even if input.close() threw
        }
    }
}
```

## Notes
- `AutoCloseable.close()` is permitted to throw any `Exception`; narrow the override's declared throws clause (`IOException` above) whenever the actual implementation only throws a specific type — this is a standard, safe override narrowing.
- Making `close()` idempotent also protects against a caller wrapping the resource in try-with-resources after already closing it manually in some other path — the second call must not re-attempt cleanup or throw.
- Prefer composing already-`AutoCloseable` resources (`InputStream`, `OutputStream` above) and closing each in its own `try`/`finally` layer so one's failure doesn't suppress attempting the next.

## References
- [AutoCloseable (Java SE 21)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/AutoCloseable.html)
- [Effective Java, 3rd Edition — Item 9: Prefer try-with-resources to try-finally](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
