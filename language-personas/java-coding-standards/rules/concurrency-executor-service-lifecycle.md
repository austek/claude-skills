---
title: Always Shut Down an ExecutorService
impact: HIGH
impactDescription: prevents non-daemon thread leaks that keep the JVM alive or exhaust thread pools
tags: [concurrency, executor-service, resource-management, lifecycle]
---

# Always Shut Down an ExecutorService [HIGH]

## Description
An `ExecutorService` owns non-daemon threads that keep running, and keep the JVM alive, until explicitly told to stop. Creating one without ever calling `shutdown()` leaks those threads for the process's lifetime — in a long-running service this exhausts thread pools over time, and in a short-lived tool it prevents the JVM from exiting. `shutdown()` alone only stops accepting new tasks; pair it with `awaitTermination(timeout, unit)` to block until in-flight tasks finish, and fall back to `shutdownNow()` if the timeout expires so stuck tasks don't hang the shutdown indefinitely. Since Java 19, `ExecutorService` extends `AutoCloseable`, and its default `close()` does exactly this sequence — prefer try-with-resources over hand-writing it.

## Bad Example
```java
public class ReportGenerator {
    private final ExecutorService executor = Executors.newFixedThreadPool(4);

    public List<Report> generate(List<Source> sources) {
        List<Future<Report>> futures = sources.stream()
            .map(source -> executor.submit(() -> build(source)))
            .toList();
        return futures.stream().map(this::join).toList();
        // executor is never shut down: its 4 threads leak for the JVM's lifetime
    }
}
```

## Good Example
```java
public class ReportGenerator {
    public List<Report> generate(List<Source> sources) {
        try (ExecutorService executor = Executors.newFixedThreadPool(4)) {
            List<Future<Report>> futures = sources.stream()
                .map(source -> executor.submit(() -> build(source)))
                .toList();
            return futures.stream().map(this::join).toList();
        } // close() shuts down and awaits termination, forcing a stop if interrupted
    }
}
```

## Notes
- On Java versions without `AutoCloseable` support in practice, shut down manually: `shutdown()`, then `awaitTermination(timeout, unit)` in a bounded wait, then `shutdownNow()` if it times out — always in a `finally` block.
- An `ExecutorService` that outlives a single method (a field on a long-lived service) needs its shutdown wired into the owning component's own lifecycle (e.g., a `@PreDestroy` method), not try-with-resources.
- `shutdownNow()` only *attempts* to cancel running tasks via interruption; tasks that ignore `InterruptedException` will not actually stop.

## References
- [ExecutorService (Java SE 21)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ExecutorService.html)
- [Java Concurrency in Practice — Ch. 7: Cancellation and Shutdown](https://jcip.net/)
