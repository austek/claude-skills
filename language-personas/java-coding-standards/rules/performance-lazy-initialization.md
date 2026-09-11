---
title: Defer Expensive Initialization Until First Use
impact: MEDIUM
impactDescription: avoids paying construction cost for a resource a given run may never touch
tags: [performance, initialization, immutability]
---

# Defer Expensive Initialization Until First Use [MEDIUM]

## Description
Eager initialization — building an expensive object (a large cache, a compiled regex set, a connection pool, a parsed configuration) in a constructor or static initializer regardless of whether that run of the program ever needs it — pays the full construction cost up front on every startup, even for code paths that never touch the value. Where the value is genuinely used on almost every execution, eager initialization is simplest and correct; the rule applies specifically when initialization is measurably expensive *and* conditionally needed — a diagnostic report generator only invoked when a flag is set, a rarely used export format, a fallback strategy exercised only when a primary path fails.

`Supplier<T>` deferred via a `java.util.function.Supplier` field, or the holder-class idiom for a thread-safe lazy singleton, both defer the cost to first use without the correctness hazards of hand-rolled double-checked locking (see `concurrency-avoid-double-checked-locking`). The trade-off is real: lazy initialization adds a branch and, in a multi-threaded context, a synchronization point on every access after the first — worth it only when the deferred cost is large and the value is conditionally needed, not as a default habit.

## Bad Example
```java
public class ReportService {
    private final ExpensiveTemplateEngine templateEngine = new ExpensiveTemplateEngine(); // built even if never used

    public byte[] generatePdfReport(ReportRequest request) {
        if (request.format() != Format.PDF) {
            return generateCsv(request); // templateEngine was still constructed for this call
        }
        return templateEngine.render(request);
    }
}
```

## Good Example
```java
public class ReportService {
    private final Supplier<ExpensiveTemplateEngine> templateEngine =
            Suppliers.memoize(ExpensiveTemplateEngine::new); // built on first use, cached after

    public byte[] generatePdfReport(ReportRequest request) {
        if (request.format() != Format.PDF) {
            return generateCsv(request); // never triggers construction
        }
        return templateEngine.get().render(request);
    }
}
```

## Notes
- The JVM's class-initialization-on-first-use semantics already give static fields inside a not-yet-loaded class this property for free — an idiom sometimes called the initialization-on-demand holder pattern relies on exactly this for a thread-safe lazy singleton with zero explicit synchronization.
- A `Supplier` field that's just wrapping cheap construction adds indirection for no benefit — apply this rule only where profiling or clear reasoning shows the deferred cost is worth the added complexity, consistent with `performance-avoid-premature-optimization`.
- `Suppliers.memoize` (Guava) and equivalents cache the computed value after first access — without memoization, a naive `Supplier` recomputes on every call, which is worse than eager initialization for a value accessed more than once.

## References
- [Effective Java, 3rd Edition — Item 83: Use lazy initialization judiciously](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
- [Guava — Suppliers.memoize](https://guava.dev/releases/snapshot-jre/api/docs/com/google/common/base/Suppliers.html)
