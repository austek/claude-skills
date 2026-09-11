---
title: Design Fluent APIs for Method Chaining Where the Domain Fits
impact: LOW
impactDescription: reads as a pipeline of intent instead of a sequence of statements referencing a shared variable
tags: [api-design, fluent-interface, readability]
---

# Design Fluent APIs for Method Chaining Where the Domain Fits [LOW]

## Description
A fluent API returns `this` (or a new instance, for an immutable type) from each configuring method, letting calls chain into one expression that reads close to a sentence describing what's being built or queried — `Stream`, `Optional`, and the builder pattern (`api-design-builder-for-many-params`) all use this shape. It earns its keep for step-by-step configuration or transformation pipelines where the order of calls has an obvious reading order. It is the wrong shape for methods that don't naturally compose in sequence, or where chaining would hide a meaningful return value (a boolean success flag, a computed result) behind a chain that looks like it's still just configuring.

## Bad Example
```java
public class QueryBuilder {
    private String table;
    private String whereClause;

    public void from(String table) { this.table = table; } // void: cannot chain
    public void where(String clause) { this.whereClause = clause; }
}

QueryBuilder builder = new QueryBuilder();
builder.from("orders");
builder.where("status = 'PENDING'");
// Three statements, a throwaway variable, no visible relationship between the calls
```

## Good Example
```java
public final class QueryBuilder {
    private final String table;
    private final String whereClause;

    private QueryBuilder(String table, String whereClause) {
        this.table = table;
        this.whereClause = whereClause;
    }

    public static QueryBuilder from(String table) {
        return new QueryBuilder(table, null);
    }

    public QueryBuilder where(String clause) {
        return new QueryBuilder(table, clause); // returns a new instance, stays immutable
    }
}

Query query = QueryBuilder.from("orders")
    .where("status = 'PENDING'")
    .build();
```

## Notes
- Prefer returning a new instance over mutating and returning `this` when the type is otherwise immutable (`immutability-*` rules) — a fluent API is a calling convention, not a license to make a value type mutable.
- Do not make a method fluent (returning `this`) purely for chaining if its real job is to report a result — `boolean isValid()` returning `this` instead of the boolean hides the answer the caller asked for.
- A chain that spans more than roughly five or six calls is often a sign the intermediate steps deserve names — break it into named local variables at that point.

## References
- [Effective Java, 3rd Edition — Item 2: Consider a builder](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
