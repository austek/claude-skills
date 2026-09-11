---
title: Pick the Right Collectors Factory for the Shape You Need
impact: MEDIUM
impactDescription: replaces hand-rolled post-processing with a single correctly-shaped collect call
tags: [streams, collectors, collections]
---

# Pick the Right Collectors Factory for the Shape You Need [MEDIUM]

## Description
`Collectors` provides purpose-built factories for the common terminal shapes a stream pipeline needs — grouping (`groupingBy`), partitioning by a boolean predicate (`partitioningBy`), building a map (`toMap`), joining strings (`joining`), and simple aggregation (`counting`, `summingInt`, `averagingDouble`). Reaching for `.collect(Collectors.toList())` and then hand-rolling the grouping or joining logic afterward reimplements what a single, more specific `collect` call already does correctly, including edge cases (duplicate keys in `toMap`, empty-stream behavior) that a manual loop is easy to get wrong.

## Bad Example
```java
public Map<Role, List<User>> groupByRole(List<User> users) {
    List<User> all = users.stream().toList();
    Map<Role, List<User>> result = new HashMap<>(); // hand-rolled grouping
    for (User user : all) {
        result.computeIfAbsent(user.getRole(), r -> new ArrayList<>()).add(user);
    }
    return result;
}

public String joinNames(List<User> users) {
    StringBuilder sb = new StringBuilder(); // hand-rolled joining
    for (int i = 0; i < users.size(); i++) {
        if (i > 0) sb.append(", ");
        sb.append(users.get(i).getName());
    }
    return sb.toString();
}
```

## Good Example
```java
public Map<Role, List<User>> groupByRole(List<User> users) {
    return users.stream()
        .collect(Collectors.groupingBy(User::getRole));
}

public String joinNames(List<User> users) {
    return users.stream()
        .map(User::getName)
        .collect(Collectors.joining(", "));
}

public Map<Boolean, List<User>> splitByActive(List<User> users) {
    return users.stream()
        .collect(Collectors.partitioningBy(User::isActive));
}

public Map<Long, String> emailById(List<User> users) {
    return users.stream()
        .collect(Collectors.toMap(User::getId, User::getEmail));
}
```

## Notes
- `Collectors.toMap(keyFn, valueFn)` throws `IllegalStateException` on a duplicate key; pass a third merge-function argument (`(a, b) -> a`) when duplicates are expected and one should win.
- `Collectors.groupingBy(classifier, downstream)` composes with a second collector — e.g., `groupingBy(User::getRole, Collectors.counting())` for a per-group count instead of a per-group list.
- `Collectors.toUnmodifiableList()`/`toUnmodifiableMap()` exist for explicit immutability when `Stream.toList()`'s default unmodifiable result isn't reachable (e.g., inside a `groupingBy` downstream collector).
- For an empty input stream, `groupingBy` returns an empty map and `joining` returns an empty string — no null checks are needed on the result.

## References
- [Collectors (Java SE 21 & JDK 21)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Collectors.html)
