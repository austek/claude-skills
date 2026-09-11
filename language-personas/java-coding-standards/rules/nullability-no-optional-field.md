---
title: Never Use Optional as a Field Type
impact: MEDIUM
impactDescription: avoids non-serializable, memory-heavier field wrappers
tags: [nullability, optional, api-design]
---

# Never Use Optional as a Field Type [MEDIUM]

## Description
`Optional<T>` is not `Serializable`, so any class holding one as a field breaks Java serialization, most JSON mapping libraries' default behavior, and JPA/Hibernate entity mapping outright. It also adds an extra object allocation to every instance for a problem the field already solves: a field is either set or it is not, and that can be expressed with a plain nullable reference plus an accessor that wraps it in `Optional` on the way out. Keep `Optional` at the boundary — the method that reads the field — not inside the object's storage.

## Bad Example
```java
public class UserProfile {
    private final String name;
    private final Optional<String> middleName; // not Serializable, extra allocation per instance

    public UserProfile(String name, Optional<String> middleName) {
        this.name = name;
        this.middleName = middleName;
    }

    public Optional<String> getMiddleName() {
        return middleName;
    }
}
```

## Good Example
```java
public class UserProfile {
    private final String name;
    private final String middleName; // nullable internally, plain and Serializable

    public UserProfile(String name, String middleName) {
        this.name = Objects.requireNonNull(name, "name");
        this.middleName = middleName;
    }

    public Optional<String> getMiddleName() {
        return Optional.ofNullable(middleName);
    }
}
```

## Notes
- Records follow the same rule: a component type should be `String`, not `Optional<String>`; expose an `Optional`-returning accessor method alongside the raw component if needed.
- If the domain genuinely models "present or absent" as a first-class concept with more than one absent reason, model it with a `sealed interface` (e.g., `Present`/`Absent`) rather than `Optional`, since `Optional` carries no reason for absence.
- JPA entities must never use `Optional` for `@Column`-mapped fields; providers do not support it as a persisted attribute type.

## References
- [Effective Java, 3rd Edition — Item 55: Return optionals judiciously](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
- [Optional (Java SE 21 & JDK 21)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Optional.html)
