---
title: Never Use Raw Generic Types
impact: HIGH
impactDescription: restores compile-time type checking that raw types silently discard
tags: [generics, type-safety, raw-types]
---

# Never Use Raw Generic Types [HIGH]

## Description
A raw type — `List` instead of `List<String>`, `Map` instead of `Map<String, Order>` — erases all compile-time type checking on that generic's contents. The compiler cannot catch a wrong-typed insertion, so the error surfaces later as a `ClassCastException` at the read site, far from where the bad value was added. Raw types exist only for backward compatibility with pre-generics (Java 4 and earlier) code; there is no reason to introduce a new raw type in code written today. Parameterize every generic type, including with a wildcard (`List<?>`) when the element type is genuinely unknown or irrelevant — a wildcard type still gives the compiler something to check, unlike a raw type, which gives it nothing.

## Bad Example
```java
List orders = new ArrayList(); // raw type: no element-type checking at all
orders.add(new Order("A-1"));
orders.add("not an order"); // compiles without error

for (Object o : orders) {
    Order order = (Order) o; // ClassCastException here, far from the actual mistake
    process(order);
}
```

## Good Example
```java
List<Order> orders = new ArrayList<>();
orders.add(new Order("A-1"));
orders.add("not an order"); // compile error: caught immediately

for (Order order : orders) {
    process(order); // no cast needed, type is enforced at every insertion point
}

// Genuinely unknown element type: use a wildcard, not a raw type
void printAll(List<?> items) {
    items.forEach(System.out::println);
}
```

## Notes
- `List<Object>` and `List<?>` are not the same as a raw `List`: both still type-check, they differ in what operations they permit (a wildcard list rejects most `add` calls, since the element type is unknown).
- Raw types produce an "unchecked call" compiler warning, not an error — do not treat the absence of a compile failure as safety.
- Static analysis tools and IDE inspections flag raw type usage by default; do not suppress that warning instead of fixing the type.

## References
- [Effective Java, 3rd Edition — Item 26: Don't use raw types](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
- [Java Generics FAQ — Raw Types](https://docs.oracle.com/javase/tutorial/java/generics/rawTypes.html)
