---
title: Never Let Secrets or PII Reach a Log Line or Span Field
impact: CRITICAL
impactDescription: A single leaked field can turn a log aggregator into a compliance incident
tags: [observability, security, tracing, pii]
---

# Never Let Secrets or PII Reach a Log Line or Span Field [CRITICAL]

## Description
Think of any value written to a log line as data that has left the application's own trust boundary: aggregator platforms (Datadog, Loki, Splunk) typically have broader internal access than the database or secrets manager the value came from, and once it's indexed there it tends to stay retrievable for a long time — in a support export, a shared debugging session, or, in the worst case, a breach disclosure the company is legally obligated to make under GDPR/HIPAA/PCI-DSS. `#[tracing::instrument]` makes this easy to trigger by accident: by default it turns every argument of the function it annotates into a captured span field, so a `Credentials` struct handed to an instrumented `authenticate` function puts its password in every trace the function emits unless something explicitly stops that from happening.

## Bad Example
```rust
struct Credentials { username: String, password: String, api_key: String }

#[tracing::instrument] // auto-captures password and api_key as span fields
async fn authenticate(credentials: &Credentials) -> bool {
    tracing::info!(?credentials, "authenticating"); // logs the whole struct, too
    true
}
```

## Good Example
```rust
#[derive(Clone)]
struct Secret(String);

impl std::fmt::Debug for Secret {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        f.write_str("[redacted]")
    }
}

struct Credentials { username: String, password: Secret, api_key: Secret }

#[tracing::instrument(skip(credentials), fields(username = %credentials.username))]
async fn authenticate(credentials: &Credentials) -> bool {
    tracing::info!("authenticating");
    verify(&credentials.username, &credentials.password)
}
# fn verify(_u: &str, _p: &Secret) -> bool { true }
```

## Notes
- `skip(arg)` removes one argument from `#[instrument]`'s auto-captured fields; `skip_all` removes every argument, after which only fields explicitly listed in `fields(...)` are recorded — the safer default for any function taking a struct that might contain sensitive fields.
- A redacting `Debug`/`Display` impl on a wrapper type protects every call site, including ones added later by someone who doesn't know the field is sensitive — the `secrecy` crate's `Secret<T>` gives this for free plus an explicit `.expose_secret()` that's the only place the real value can leak out.
- Full request/response bodies are a common accidental leak vector — a JSON payload can carry an auth token or PII nested arbitrarily deep, so log request metadata (path, size, content-type) instead of the body itself in production.

## References
- [obs-instrument-spans](obs-instrument-spans.md)
- [obs-structured-fields](obs-structured-fields.md)
- [err-thiserror-lib](err-thiserror-lib.md)
