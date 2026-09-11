---
title: Prefer Primitive Arrays Over Boxed Collections in Verified Hot Paths
impact: MEDIUM
impactDescription: eliminates per-element object overhead and pointer-chasing in numeric-heavy inner loops
tags: [performance, collections-streams, primitives]
---

# Prefer Primitive Arrays Over Boxed Collections in Verified Hot Paths [MEDIUM]

## Description
A `List<Integer>` stores boxed `Integer` objects scattered across the heap, each with object-header overhead (typically 16 bytes on a modern JVM before the 4-byte `int` payload) and accessed through a layer of reference indirection — iterating it means chasing a pointer to each boxed value rather than reading a contiguous block of memory. An `int[]` stores the raw values contiguously, with no per-element allocation and cache-friendly sequential access. For a hot numeric loop already confirmed by profiling to be a bottleneck — a matrix operation, a tight aggregation over millions of values — that difference in memory layout and allocation is a large, measurable share of the method's cost, on top of the boxing/unboxing overhead covered separately in `performance-avoid-autoboxing-hot-paths`.

This is not a general recommendation to prefer arrays over `List`/`Set`/`Map` in ordinary code — collections offer resizing, a much richer API, and integration with the rest of the standard library that a raw array doesn't, and that convenience is worth far more than the allocation difference in code that isn't a measured hot path. Reach for a primitive array only after profiling has identified the specific loop where it matters.

## Bad Example
```java
public double sumSquares(List<Double> values) {
    // each element is a separate heap object; the loop chases a reference per element
    double total = 0.0;
    for (Double v : values) {
        total += v * v;
    }
    return total;
}
```

## Good Example
```java
public double sumSquares(double[] values) {
    // contiguous primitive storage; no per-element allocation, cache-friendly access
    double total = 0.0;
    for (double v : values) {
        total += v * v;
    }
    return total;
}
```

## Notes
- Converting between a `List<Double>` and a `double[]` at an API boundary (`values.stream().mapToDouble(Double::doubleValue).toArray()`) lets the public API stay collection-based while the verified-hot inner computation works on primitives.
- Arrays lack the safety and API richness of collections — no built-in bounds-checked resize, no `Optional`-returning lookups, and array covariance famously allows a runtime `ArrayStoreException` that generics prevent at compile time — none of which matters for a private, tightly scoped hot loop but all of which matter for a public API surface.
- This rule and `performance-avoid-autoboxing-hot-paths` are closely related — a boxed `List<Integer>` pays both an allocation cost per box and an indirection cost per access, while a primitive array pays neither.

## References
- [Java Language Specification — Arrays](https://docs.oracle.com/javase/specs/jls/se21/html/jls-10.html)
- [Effective Java, 3rd Edition — Item 6: Avoid creating unnecessary objects](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
