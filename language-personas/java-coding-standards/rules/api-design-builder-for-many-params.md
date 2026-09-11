---
title: Use a Builder When a Constructor Needs Many Parameters
impact: MEDIUM
impactDescription: eliminates positional-argument mistakes and optional-parameter overload explosion
tags: [api-design, builder, constructors, readability]
---

# Use a Builder When a Constructor Needs Many Parameters [MEDIUM]

## Description
A constructor with more than four or five parameters, especially several of the same type (two `String`s, three `int`s) or several optional ones, is easy to call with arguments in the wrong order — the compiler accepts `new Request(url, method, 30, true)` regardless of whether `30` and `true` land in the right positions. Telescoping constructors (one overload per combination of optional parameters) scale combinatorially and still don't name what each argument means at the call site. A builder names every parameter at the point it's set, handles optional parameters as calls that can simply be omitted, and validates the complete, assembled state once in a single `build()` step rather than across N constructor overloads.

## Bad Example
```java
public class HttpRequest {
    public HttpRequest(String url, String method, int timeoutMs, boolean followRedirects,
                        boolean retryOnFailure, int maxRetries) { /* ... */ }
}

// Which boolean is which? Order-dependent, unreadable at the call site.
new HttpRequest("https://api.example.com", "GET", 5000, true, true, 3);
```

## Good Example
```java
public final class HttpRequest {
    private HttpRequest(Builder builder) { /* assign fields from builder */ }

    public static Builder builder(String url, String method) {
        return new Builder(url, method);
    }

    public static final class Builder {
        private final String url;
        private final String method;
        private int timeoutMs = 3000;
        private boolean followRedirects = true;

        private Builder(String url, String method) {
            this.url = url;
            this.method = method;
        }

        public Builder timeoutMs(int timeoutMs) { this.timeoutMs = timeoutMs; return this; }
        public Builder followRedirects(boolean value) { this.followRedirects = value; return this; }
        public HttpRequest build() { return new HttpRequest(this); }
    }
}

HttpRequest request = HttpRequest.builder("https://api.example.com", "GET")
    .timeoutMs(5000)
    .followRedirects(false)
    .build();
```

## Notes
- Keep genuinely required parameters (`url`, `method` above) on the builder's factory method, not as no-arg-default builder setters — this makes omitting them a compile error, not a runtime surprise.
- For a data-only type with no required construction-order logic, prefer a record with named factory methods over a full builder — reach for the builder pattern once there are several optional fields or construction needs validation across them.
- `build()` is the natural place to validate invariants that span multiple fields, since it's the one point where the full parameter set is known.

## References
- [Effective Java, 3rd Edition — Item 2: Consider a builder when faced with many constructor parameters](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
