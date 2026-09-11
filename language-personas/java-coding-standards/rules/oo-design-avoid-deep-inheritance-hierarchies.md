---
title: Avoid Deep Inheritance Hierarchies
impact: MEDIUM
impactDescription: understanding one class no longer requires tracing behavior up several superclasses
tags: [oo-design, inheritance, complexity, maintainability]
---

# Avoid Deep Inheritance Hierarchies [MEDIUM]

## Description
Every extra level in an inheritance chain is another place a field can be shadowed, a method can be overridden with subtly different behavior, or a constructor can do something a subclass three levels down didn't anticipate. Understanding what a deeply nested class actually does means tracing behavior up through every superclass in the chain, since any of them can contribute fields, override points, or side effects at construction. This cost compounds with depth in a way flat composition does not — a class composed of three collaborators only requires understanding those three contracts, not an inheritance chain none of whose links can change independently. Keep hierarchies shallow (two to three levels is a reasonable ceiling) and prefer composition (`oo-design-composition-over-inheritance`) once a hierarchy would need to go deeper to express a new distinction.

## Bad Example
```java
class Vehicle { /* ... */ }
class MotorVehicle extends Vehicle { /* ... */ }
class Car extends MotorVehicle { /* ... */ }
class SportsCar extends Car { /* ... */ }
class ConvertibleSportsCar extends SportsCar { /* ... */ }
// Understanding ConvertibleSportsCar means reading five classes' worth
// of fields, overrides, and constructor behavior to know what it actually does.
```

## Good Example
```java
// Flat: one level of inheritance for the genuine is-a relationship,
// the rest expressed as composed capabilities.
sealed interface Vehicle permits MotorVehicle {}

record MotorVehicle(Engine engine, BodyStyle bodyStyle, RoofType roofType) implements Vehicle {}

enum BodyStyle { SEDAN, SPORTS }
enum RoofType { FIXED, CONVERTIBLE }

MotorVehicle convertibleSportsCar = new MotorVehicle(
    new Engine(EngineType.V8), BodyStyle.SPORTS, RoofType.CONVERTIBLE);
```

## Notes
- A hierarchy that is only ever discriminated by `switch`/`instanceof` at its leaves, rather than by behavior differing per level, is usually better modeled as a sealed interface with flat, data-only variants than as deep inheritance.
- Depth is a smell most clearly when intermediate classes exist only to be extended further and are never instantiated directly — that is a sign the "is-a" relationship they express is not actually meaningful on its own.
- Two to three levels is a guideline, not a hard limit — the actual signal is whether tracing a class's behavior up its chain is still easy to hold in mind.

## References
- [Effective Java, 3rd Edition — Item 18: Favor composition over inheritance](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
