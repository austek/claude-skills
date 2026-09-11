---
title: Extract Cross-Cutting Setup into a JUnit Extension
impact: MEDIUM
impactDescription: eliminates copy-pasted @BeforeEach bodies across every test class that needs the same setup
tags: [junit5, extensions, fixtures, reuse]
---

# Extract Cross-Cutting Setup into a JUnit Extension [MEDIUM]

## Description
JUnit 5's `Extension` API lets setup and teardown logic that several test classes need be written once and applied with `@ExtendWith(MyExtension.class)`, instead of copy-pasted into every class's `@BeforeEach`/`@AfterEach`. The relevant callback interfaces — `BeforeEachCallback`, `AfterEachCallback`, `BeforeAllCallback`, `AfterAllCallback`, `ParameterResolver`, `TestInstancePostProcessor` — cover the same lifecycle points as the annotations, but package the behavior as a reusable, independently testable unit. A `ParameterResolver` in particular lets an extension inject a fixture directly as a test method parameter, which keeps the test signature declaring exactly what it needs.

The point where a `@BeforeEach` method stops being "this test class's setup" and starts being "infrastructure every test class in the module needs" is the point to extract an extension: a WireMock server, a migrated in-memory database, a fixed `Clock`. Left as copy-pasted `@BeforeEach` bodies, that setup drifts — one class updates its copy for a bug fix, the others don't — and every class carries boilerplate unrelated to what it's actually testing. An extension also composes: `@ExtendWith` accepts multiple extensions, each independently reusable, rather than one growing `@BeforeEach` method doing several unrelated things.

Reach for an extension specifically for setup that's identical across classes. Setup that varies per test class — building that class's particular subject under test — belongs in that class's own `@BeforeEach` or a test-data builder (`fixtures-builder-for-test-data`), not in an extension built to hide the variation.

## Bad Example
```java
class OrderServiceTest {
    WireMockServer wireMock;

    @BeforeEach
    void startWireMock() {
        wireMock = new WireMockServer(0);
        wireMock.start();
    }

    @AfterEach
    void stopWireMock() {
        wireMock.stop();
    }
    // identical block copy-pasted into PaymentServiceTest, ShippingServiceTest, ...
}
```

## Good Example
```java
class WireMockExtension implements BeforeEachCallback, AfterEachCallback, ParameterResolver {
    private WireMockServer server;

    @Override
    public void beforeEach(ExtensionContext context) {
        server = new WireMockServer(0);
        server.start();
    }

    @Override
    public void afterEach(ExtensionContext context) {
        server.stop();
    }

    @Override
    public boolean supportsParameter(ParameterContext pc, ExtensionContext ec) {
        return pc.getParameter().getType() == WireMockServer.class;
    }

    @Override
    public Object resolveParameter(ParameterContext pc, ExtensionContext ec) {
        return server;
    }
}

@ExtendWith(WireMockExtension.class)
class OrderServiceTest {
    @Test
    void fetchesShippingRate(WireMockServer wireMock) {
        wireMock.stubFor(get("/rates").willReturn(okJson("{\"rate\":5.00}")));
        // ...
    }
}
```

## Notes
- Register an extension declaratively with `@ExtendWith` on the class (or a meta-annotation combining several) rather than `RegisterExtension` unless the test needs to configure the instance per class.
- Extensions compose with `@BeforeEach`/`@AfterEach` (`junit5-structure-lifecycle-annotations`): use an extension for setup shared across classes, plain lifecycle annotations for setup local to one class.
- An extension that starts a real out-of-process resource (a container, a server) should pair with explicit lifecycle scoping — see `testcontainers-lifecycle-scope` for the same concern applied to Testcontainers.

## References
- [JUnit 5 User Guide — Extension Model](https://junit.org/junit5/docs/current/user-guide/#extensions)
