---
title: Export Only Packages That Are Part of the Public API
impact: HIGH
impactDescription: turns "don't depend on our internals" from a documentation request into a compile error
tags: [modules, jpms, encapsulation, api-design]
---

# Export Only Packages That Are Part of the Public API [HIGH]

## Description
The Java Platform Module System (JPMS, Java 9+) makes a package invisible to other modules by default — a package not named in an `exports` clause of `module-info.java` cannot be compiled against or, without a reflective `--add-opens` escape hatch, accessed at all from outside the module, regardless of the visibility modifiers on its classes. That's strictly stronger than the pre-module convention of naming a package `internal` or `impl` and hoping nobody imports it — a `public` class in an unexported package is `public` only within its own module, so accidental coupling to an implementation detail becomes a build failure at the consumer, not a runtime surprise discovered later.

Exporting a package is also close to a one-way door: once another module compiles against it, removing the export is a breaking change for every consumer. Deciding upfront which packages are the actual API — versus internal helpers, wiring classes, or vendored code — and exporting only those keeps the module's real surface area small and its freedom to refactor internals intact.

## Bad Example
```java
module com.example.billing {
    exports com.example.billing.api;
    exports com.example.billing.internal; // implementation detail, now a committed public surface
    exports com.example.billing.internal.jdbc;
}
```

## Good Example
```java
module com.example.billing {
    exports com.example.billing.api;

    requires java.sql;
    // com.example.billing.internal and .internal.jdbc are not exported —
    // they compile and run fine within this module but are invisible outside it
}
```

## Notes
- `exports ... to` (a qualified export) restricts visibility to named modules — useful for a package that a sibling module needs but that shouldn't be part of the general public API.
- `opens`/`opens ... to` grants reflective access without compile-time visibility, which is what frameworks like Jackson or Hibernate need for reflection-based (de)serialization on an otherwise unexported package — see `reflection-serialization-jackson-explicit-config` and `reflection-serialization-avoid-field-injection` for why relying on reflection into internals is itself worth minimizing.
- Not every project needs JPMS modules — a library published for wide consumption benefits most; an internal application module with a single consumer gets less from the mechanism than from simply keeping fewer packages public.

## References
- [JEP 261: Module System](https://openjdk.org/jeps/261)
- [Java Platform Module System — Oracle documentation](https://docs.oracle.com/javase/9/docs/api/java/lang/module/package-summary.html)
