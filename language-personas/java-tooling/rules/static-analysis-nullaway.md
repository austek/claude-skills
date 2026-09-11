---
title: Enforce Default Non-Null with NullAway, Not Just Objects.requireNonNull
impact: HIGH
impactDescription: catches a null-dereference path at compile time instead of a runtime NullPointerException
tags: [nullaway, error-prone, static-analysis, nullability]
---

# Enforce Default Non-Null with NullAway, Not Just Objects.requireNonNull [HIGH]

## Description
NullAway (Uber) is an Error Prone check that performs a fast, type-based nullability analysis: every parameter, field, and return type is treated as non-null by default unless explicitly annotated `@Nullable`, and NullAway flags any code path that could dereference a `@Nullable`-annotated value without a preceding null check, or that returns/passes a possibly-null value into a non-null-typed slot. It deliberately trades soundness for speed and a low false-positive rate against heavier interprocedural tools like the Checker Framework, which is what lets it run inside the normal compile step on every build instead of as a separate, slower analysis pass developers skip locally.

Because it defaults every unannotated type to non-null, turning NullAway on repo-wide against an existing, unannotated codebase produces a wall of findings on day one and stalls adoption. `-XepOpt:NullAway:AnnotatedPackages=com.example.billing` scopes enforcement to specific packages, so a team brings modules under strict null-checking one at a time rather than all at once.

NullAway requires Error Prone already wired into the build (`static-analysis-error-prone-compiler-plugin.md`), since it ships as one of Error Prone's checks rather than a standalone javac plugin — there's no separate compiler integration to configure beyond enabling this one check.

## Bad Example
```java
Map<String, Customer> customers = ...;

Customer lookup(String id) {
    return customers.get(id); // can return null — no annotation says so, no check follows
}

void charge(String id) {
    lookup(id).charge(); // compiles cleanly, NPEs when id is missing
}
```

## Good Example
```java
@Nullable
Customer lookup(String id) {
    return customers.get(id);
}

void charge(String id) {
    Customer customer = lookup(id);
    if (customer == null) {
        throw new CustomerNotFoundException(id);
    }
    customer.charge(); // NullAway confirms this path is unreachable when null
}
```

## Notes
- NullAway only reasons about what's visible at the type level within annotated packages — it doesn't replace `../../java-coding-standards/rules/nullability-objects-requirenonnull.md` at boundaries where a value crosses into unannotated third-party code NullAway can't see into.
- Any standard `@Nullable` annotation (JSR-305-style, from `javax.annotation`, `org.jspecify`, etc.) is recognized — NullAway does not require its own annotation type.
- `-XepOpt:NullAway:TreatGeneratedAsUnannotated=true` avoids false positives on generated sources (protobuf, Lombok) that predictably lack nullability annotations.

## References
- [NullAway — GitHub](https://github.com/uber/NullAway)
- [NullAway Wiki — Adopting NullAway in an Existing Codebase](https://github.com/uber/NullAway/wiki/Configuration)
