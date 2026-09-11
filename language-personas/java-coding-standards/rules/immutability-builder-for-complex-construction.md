---
title: Use the Builder Pattern for Objects with Many Optional Fields
impact: MEDIUM
impactDescription: replaces error-prone telescoping constructors with a readable, safe assembly step
tags: [immutability, builder, api-design]
---

# Use the Builder Pattern for Objects with Many Optional Fields [MEDIUM]

## Description
An immutable class with several optional fields either forces callers through a "telescoping constructor" — a wall of overloaded constructors covering every combination of optional arguments — or, worse, exposes setters that reintroduce mutability just to make construction convenient. The builder pattern keeps the target object fully immutable while giving callers a readable, named-parameter-like way to assemble it: a separate (typically mutable, or record-with-`with`-methods) builder object accumulates values, and a single `build()` call produces the final immutable instance, validating invariants at that one point.

Reach for a builder only once a constructor would need four or more parameters, several of which are optional — for two or three fields, a plain constructor or a compact record is simpler and should be preferred.

## Bad Example
```java
public final class HttpRequest {
    private final String url;
    private final String method;
    private final Map<String, String> headers;
    private final byte[] body;
    private final Duration timeout;

    // Telescoping constructors: every new optional combination adds another overload
    public HttpRequest(String url, String method) {
        this(url, method, Map.of(), null, Duration.ofSeconds(30));
    }

    public HttpRequest(String url, String method, Map<String, String> headers) {
        this(url, method, headers, null, Duration.ofSeconds(30));
    }

    public HttpRequest(String url, String method, Map<String, String> headers, byte[] body, Duration timeout) {
        this.url = url;
        this.method = method;
        this.headers = headers;
        this.body = body;
        this.timeout = timeout;
    }
}
```

## Good Example
```java
public final class HttpRequest {
    private final String url;
    private final String method;
    private final Map<String, String> headers;
    private final byte[] body;
    private final Duration timeout;

    private HttpRequest(Builder builder) {
        this.url = Objects.requireNonNull(builder.url, "url");
        this.method = builder.method;
        this.headers = Map.copyOf(builder.headers);
        this.body = builder.body == null ? null : builder.body.clone();
        this.timeout = builder.timeout;
    }

    public static Builder builder(String url) {
        return new Builder(url);
    }

    public static final class Builder {
        private final String url;
        private String method = "GET";
        private final Map<String, String> headers = new HashMap<>();
        private byte[] body;
        private Duration timeout = Duration.ofSeconds(30);

        private Builder(String url) {
            this.url = url;
        }

        public Builder method(String method) { this.method = method; return this; }
        public Builder header(String key, String value) { headers.put(key, value); return this; }
        public Builder body(byte[] body) { this.body = body; return this; }
        public Builder timeout(Duration timeout) { this.timeout = timeout; return this; }

        public HttpRequest build() {
            return new HttpRequest(this); // validation and copying happen here, once
        }
    }
}

HttpRequest request = HttpRequest.builder("https://api.example.com/users")
    .method("POST")
    .header("Content-Type", "application/json")
    .timeout(Duration.ofSeconds(10))
    .build();
```

## Notes
- The built target class must still follow `immutability-final-fields` and `immutability-defensive-copy` — the builder solves ergonomics, not the target's own immutability.
- For a fixed, moderate number of optional fields, consider a record with a canonical constructor plus static factory helpers before reaching for a full builder — builders earn their weight past roughly four optional fields or when construction needs multi-step validation.
- `build()` is the single place to enforce cross-field invariants (e.g., "end must be after start") — validating inside individual builder setter methods is premature, since the object is not complete yet.

## References
- [Effective Java, 3rd Edition — Item 2: Consider a builder when faced with many constructor parameters](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
