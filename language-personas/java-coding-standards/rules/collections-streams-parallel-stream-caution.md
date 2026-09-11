---
title: Reserve parallelStream() for CPU-Bound, Sizable, Independent Work
impact: MEDIUM
impactDescription: prevents thread-pool starvation and net slowdowns from misapplied parallelism
tags: [streams, parallelism, performance, thread-safety]
---

# Reserve parallelStream() for CPU-Bound, Sizable, Independent Work [MEDIUM]

## Description
`parallelStream()` runs the pipeline on the common `ForkJoinPool`, the same pool the whole JVM shares for parallel streams and, in most applications, for other fork/join work too. It only pays off when three conditions hold together: the per-element work is CPU-bound (not blocked on I/O — a blocking call inside a parallel stream can starve that shared pool for unrelated code), the collection is large enough that split/merge overhead is smaller than the work saved, and the operations are genuinely independent with no shared mutable state (see `collections-streams-no-side-effects` — a parallel stream makes accidental shared mutation a race condition, not just unspecified ordering). Without measuring against the sequential baseline, `parallelStream()` is as likely to make things slower as faster: for a small collection, or for cheap per-element work, the coordination overhead outweighs any gain.

## Bad Example
```java
public List<String> fetchAllProfiles(List<Long> userIds) {
    return userIds.parallelStream() // I/O-bound work on the shared ForkJoinPool
        .map(id -> httpClient.get("/users/" + id)) // blocks a common-pool thread per call
        .toList();
}

public int sumSmallList(List<Integer> values) { // values typically has 5-10 elements
    return values.parallelStream()
        .mapToInt(Integer::intValue)
        .sum(); // split/merge overhead exceeds any gain at this size
}
```

## Good Example
```java
public List<String> fetchAllProfiles(List<Long> userIds) {
    // I/O-bound: use a dedicated async executor, not parallelStream, to avoid starving the common pool
    List<CompletableFuture<String>> futures = userIds.stream()
        .map(id -> CompletableFuture.supplyAsync(() -> httpClient.get("/users/" + id), ioExecutor))
        .toList();
    return futures.stream().map(CompletableFuture::join).toList();
}

public BigDecimal revalueLargePortfolio(List<Position> positions) {
    // CPU-bound, large, independent per-element computation: parallelStream is appropriate here
    return positions.parallelStream()
        .map(Position::currentMarketValue) // pure function, no shared state
        .reduce(BigDecimal.ZERO, BigDecimal::add);
}
```

## Notes
- Measure before and after with a realistic input size — `parallelStream()`'s benefit is workload- and hardware-dependent, never assume it from the API name alone.
- Never run blocking I/O inside a `parallelStream()` pipeline; it consumes common-pool threads that unrelated parallel streams elsewhere in the JVM (including in framework/library code) depend on.
- `reduce`'s combiner function must be associative for correct parallel results — the same reduction that works sequentially can produce wrong answers in parallel if the combiner isn't truly associative.
- Ordered operations (`sorted`, `limit` on an ordered stream) reduce or eliminate the benefit of parallelism because the runtime must preserve encounter order; consider `unordered()` when order genuinely does not matter.

## References
- [Parallelism (The Java Tutorials — Aggregate Operations)](https://docs.oracle.com/javase/tutorial/collections/streams/parallelism.html)
- [Effective Java, 3rd Edition — Item 48: Use caution when making streams parallel](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
