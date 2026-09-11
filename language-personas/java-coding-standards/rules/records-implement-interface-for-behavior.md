---
title: Implement an Interface on a Record to Add Shared Behavior
impact: MEDIUM
impactDescription: keeps records interchangeable with other implementations wherever the behavior is needed
tags: [records, interfaces, api-design, sealed-classes]
---

# Implement an Interface on a Record to Add Shared Behavior [MEDIUM]

## Description
A record cannot extend a class (it implicitly extends `Record`), but it can implement any number of interfaces, which is the correct route for giving a group of records a shared contract or shared default behavior. This is also how records participate in the sealed-hierarchy pattern (`sealed-pattern-*` rules): a sealed interface's `permits` implementations are commonly records, each supplying the interface's abstract methods with its own component data while inheriting any `default` methods for free. Prefer this over duplicating the same instance method across several records, or writing a static utility method that takes the record as a parameter — the interface states the shared contract once and lets `switch`/polymorphic dispatch use it directly.

## Bad Example
```java
public record Circle(double radius) {}
public record Rectangle(double width, double height) {}

// area() duplicated as an external static method per shape,
// with no shared contract tying them together
public class Shapes {
    public static double area(Circle c) { return Math.PI * c.radius() * c.radius(); }
    public static double area(Rectangle r) { return r.width() * r.height(); }
}
```

## Good Example
```java
public sealed interface Shape permits Circle, Rectangle {
    double area();

    default String describe() {
        return "%s with area %.2f".formatted(getClass().getSimpleName(), area());
    }
}

public record Circle(double radius) implements Shape {
    public double area() { return Math.PI * radius * radius; }
}

public record Rectangle(double width, double height) implements Shape {
    public double area() { return width * height; }
}

// Every Shape gets describe() for free, and area() is called polymorphically
List<Shape> shapes = List.of(new Circle(2), new Rectangle(3, 4));
shapes.forEach(shape -> System.out.println(shape.describe()));
```

## Notes
- An interface method implemented in a record must not collide with an implicitly generated accessor of the same name and no-arg signature — a component named `area` combined with an `area()` interface method would conflict, so name components and behavior methods distinctly.
- A record can implement multiple interfaces at once; this is a natural way to compose, e.g., a `Comparable<T>` contract alongside a domain-specific sealed interface.
- A `default` method on the interface (`describe()` above) runs identically for every implementing record, unless a specific record chooses to override it.

## References
- [JEP 395: Records](https://openjdk.org/jeps/395)
- [JEP 409: Sealed Classes](https://openjdk.org/jeps/409)
