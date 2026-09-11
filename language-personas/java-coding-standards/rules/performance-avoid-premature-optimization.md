---
title: Measure Before Optimizing
impact: MEDIUM
impactDescription: redirects optimization effort at the code that's actually slow instead of code that merely looks slow
tags: [performance, profiling, readability]
---

# Measure Before Optimizing [MEDIUM]

## Description
Optimizing code before a profiler or benchmark has shown it matters trades away readability and maintainability for a performance gain that's usually either imaginary or irrelevant to the program's actual bottleneck — Knuth's often-cited observation that premature optimization is the root of much evil in programming holds because intuition about hot spots is unreliable: the method a developer assumes is slow is frequently not where time is actually spent, while the real bottleneck (a synchronous network call, an N+1 query, a missing index) is invisible to code-level reasoning and only shows up under measurement. The cost isn't just wasted effort — a hand-unrolled loop or a cache added "just in case" adds code paths and state that every future reader has to understand and every future change has to keep correct, for a gain nobody has verified exists.

The default should be clear, idiomatic code first, followed by profiling (JFR, async-profiler, a JMH microbenchmark for a specific hot method) once there's a real performance target or complaint, and optimization aimed precisely at what the measurement identifies — not at whatever looked inefficient on a first read.

## Bad Example
```java
public List<String> activeUserNames(List<User> users) {
    // "optimized" with a manual index loop and a preallocated array on a guess,
    // before any profiling showed this method matters at all
    int n = users.size();
    String[] names = new String[n];
    int count = 0;
    for (int i = 0; i < n; i++) {
        User u = users.get(i);
        if (u.isActive()) {
            names[count++] = u.name();
        }
    }
    return Arrays.asList(Arrays.copyOf(names, count));
}
```

## Good Example
```java
public List<String> activeUserNames(List<User> users) {
    return users.stream()
            .filter(User::isActive)
            .map(User::name)
            .toList();
}
// clear and idiomatic; revisited with a JMH benchmark only if profiling
// identifies this method as an actual hot path
```

## Notes
- "Premature" is about sequencing, not about skipping performance work forever — a genuinely hot path identified by measurement is exactly where rules like `performance-avoid-autoboxing-hot-paths` and `performance-array-vs-collection-hot-path` apply.
- A `Stream` pipeline's overhead versus a hand-written loop is real but usually not the bottleneck in application code — see `collections-streams-prefer-stream-pipeline` for when the readability trade favors streams by default.
- JMH (Java Microbenchmark Harness) exists specifically because naive hand-timed loops are misled by JIT warm-up, dead-code elimination, and constant-folding — a micro-optimization decision made without it is often measuring noise.

## References
- [Donald Knuth — "Structured Programming with go to Statements" (1974), the origin of the premature-optimization observation](https://dl.acm.org/doi/10.1145/356635.356640)
- [JMH — Java Microbenchmark Harness](https://github.com/openjdk/jmh)
