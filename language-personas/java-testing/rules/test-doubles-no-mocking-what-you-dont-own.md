---
title: Don't Mock Types You Don't Own
impact: HIGH
impactDescription: eliminates tests that pass against an invented contract the real library never honors
tags: [test-doubles, mockito, third-party, boundaries]
---

# Don't Mock Types You Don't Own [HIGH]

## Description
Mocking a third-party type — an SDK class, a library interface, a JDK type — makes the test's behavior depend on a contract the test author invented by guessing, not on the real library's actual behavior. `when(httpClient.send(any(), any())).thenReturn(mockResponse)` proves only that the code under test handles whatever the test told the mock to return; it proves nothing about whether that's what the real `HttpClient` actually returns for a given request, what exceptions it actually throws, or what edge cases (redirects, retries, connection resets) it actually exposes. A refactor of the library, or simply a wrong assumption about its behavior baked into the mock from day one, leaves the test green while the integration is broken.

The fix is to own a thin wrapper interface around the third-party type — an interface your codebase defines, with a method shaped around what your code actually needs (`PaymentGateway.charge(amount)` instead of the SDK's raw `StripeClient.createCharge(...)`) — and mock that owned interface instead. The wrapper's own implementation is thin enough to be covered by a small number of integration tests (or Testcontainers-backed tests, `testcontainers-lifecycle-scope`, where the third party has a containerizable form) that actually exercise the real library, while the rest of the codebase's unit tests mock the owned interface with confidence, since its contract is something the team actually controls and can keep accurate.

This is `mockito-mock-boundaries-only` applied specifically to third-party code: the boundary to mock is the seam your code defines at the edge of the library, not the library's own types crossing into your test.

## Bad Example
```java
@Mock S3Client s3Client; // library type — its real contract isn't under the test's control

@Test
void uploadsReport() {
    when(s3Client.putObject(any(PutObjectRequest.class), any(RequestBody.class)))
        .thenReturn(PutObjectResponse.builder().eTag("etag-1").build());
    // proves nothing about whether this call shape matches what S3Client actually expects
}
```

## Good Example
```java
interface ReportStorage {           // owned interface, shaped around what the code needs
    void upload(String key, byte[] content);
}

class S3ReportStorage implements ReportStorage { /* thin wrapper, covered by its own integration test */ }

@Mock ReportStorage reportStorage;  // owned type — mocking it is safe

@Test
void uploadsReport() {
    service.publish(report);

    verify(reportStorage).upload(eq("report-1.pdf"), any());
}
```

## Notes
- The wrapper interface should be shaped around your code's needs, not mirror the SDK's full surface — a narrow interface is both easier to fake/mock and easier to keep accurate.
- This applies to JDK types too: mocking `java.time.Clock` is fine (it's designed to be substituted), but mocking something like `java.sql.ResultSet` in detail usually signals the data-access code needs its own owned abstraction instead.
- The wrapper's own tests are where `testcontainers-container-reuse` or a real sandbox/test account for the third-party service belongs — that's the one place the real contract must actually be exercised.

## References
- [Growing Object-Oriented Software, Guided by Tests — "Don't Mock Types You Don't Own"](http://www.growing-object-oriented-software.com/)
