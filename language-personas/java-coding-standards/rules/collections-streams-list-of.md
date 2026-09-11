---
title: Prefer List.of/Set.of/Map.of Over Mutable Factory Methods
impact: MEDIUM
impactDescription: fixed-size, immutable collections with less allocation than ArrayList/HashMap
tags: [collections, immutability, api-design]
---

# Prefer List.of/Set.of/Map.of Over Mutable Factory Methods [MEDIUM]

## Description
When a collection's contents are known upfront and are never meant to grow, shrink, or be reassigned, build it with `List.of(...)`, `Set.of(...)`, or `Map.of(...)` (Java 9+) instead of `new ArrayList<>(Arrays.asList(...))` or `Collections.unmodifiableList(new ArrayList<>(...))`. The `of` factories return immutable, usually more compactly-represented instances in one expression, reject `null` elements immediately instead of allowing them to surface as a `NullPointerException` later, and make the "this never changes" intent visible at the point of construction rather than relying on a reader to trace whether the reference escapes. Reach for `new ArrayList<>()`/`new HashMap<>()` only when the collection is actually going to be mutated after creation.

## Bad Example
```java
public class RoleRegistry {
    // Mutable by construction, even though the content never changes
    private static final List<String> ADMIN_ROLES =
        Collections.unmodifiableList(new ArrayList<>(Arrays.asList("OWNER", "ADMIN", "SUPERUSER")));

    private static final Map<String, Integer> ROLE_RANK = new HashMap<>();
    static {
        ROLE_RANK.put("OWNER", 3);
        ROLE_RANK.put("ADMIN", 2);
        ROLE_RANK.put("SUPERUSER", 1);
    }
}
```

## Good Example
```java
public class RoleRegistry {
    private static final List<String> ADMIN_ROLES = List.of("OWNER", "ADMIN", "SUPERUSER");

    private static final Map<String, Integer> ROLE_RANK =
        Map.of("OWNER", 3, "ADMIN", 2, "SUPERUSER", 1);
}
```

## Notes
- `Map.of` becomes unwieldy past roughly 10 entries (no varargs overload beyond key/value pairs); use `Map.ofEntries(Map.entry(k, v), ...)` for larger fixed maps.
- `List.of`/`Set.of`/`Map.of` throw `NullPointerException` on any `null` element, key, or value — this is a feature, not a limitation, since it surfaces the bug at construction instead of at first use.
- `Set.of` and `Map.of` do not guarantee iteration order; use `List.of` (which does preserve order) or a `LinkedHashMap` copy when order matters.
- To make an immutable copy of a collection you don't control the origin of, use `List.copyOf(source)` rather than `List.of(source.toArray())` — see `immutability-unmodifiable-collections`.

## References
- [List.of (Java SE 21 & JDK 21)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/List.html#of())
- [JEP 269: Convenience Factory Methods for Collections](https://openjdk.org/jeps/269)
