---
title: Declare Gradle Dependencies as api or implementation Deliberately
impact: HIGH
impactDescription: stops a transitive dependency leak from becoming every consumer's accidental compile-time coupling
tags: [modules, gradle, build-configuration, encapsulation]
---

# Declare Gradle Dependencies as api or implementation Deliberately [HIGH]

## Description
Gradle's `java-library` plugin splits dependency declarations into `api` and `implementation`: an `api` dependency is exposed on the module's own compile classpath *and* on every consumer's compile classpath transitively, while an `implementation` dependency is used to build the module but hidden from consumers entirely. Declaring everything as `api` — or using the legacy, undifferentiated `compile` configuration — means every dependency your module happens to use becomes part of every downstream module's compile-time surface, whether or not it's genuinely part of your public contract. That leaks version constraints (a consumer now can't upgrade that library independently without risking a binary-incompatible combination) and turns an internal implementation swap (say, replacing one JSON library with another) into a breaking change for every module that transitively compiled against the old one's types.

The rule of thumb: a dependency is `api` only if a type from it appears in your module's own public method signatures, fields, or superclasses/interfaces — anything a caller must also have on their classpath to use your API at all. Everything else, including a dependency used entirely inside method bodies, is `implementation`.

## Bad Example
```groovy
dependencies {
    api 'com.fasterxml.jackson.core:jackson-databind:2.17.0' // used only inside method bodies
    api 'com.google.guava:guava:33.0.0-jre'                  // used only inside method bodies
}
```
Every consumer of this module now compiles against Jackson and Guava too, even if their code never references either — and can't independently choose a different JSON library or Guava version without risking a clash.

## Good Example
```groovy
dependencies {
    implementation 'com.fasterxml.jackson.core:jackson-databind:2.17.0'
    implementation 'com.google.guava:guava:33.0.0-jre'
    api project(':billing-model') // Invoice, a type from this module, appears in our public methods
}
```

## Notes
- Switching an existing `api` dependency to `implementation` is a breaking change for any consumer that was (even accidentally) relying on the transitive exposure — audit consumers before tightening visibility on a published library.
- `compileOnly` and `runtimeOnly` cover annotation-processor-only and runtime-only dependencies respectively, neither of which belongs on the `api`/`implementation` axis at all.
- This is a build-tool-level analogue of `modules-jpms-explicit-exports` — both are about declaring true surface area explicitly rather than defaulting to maximal exposure.

## References
- [Gradle User Guide — The Java Library Plugin, api vs implementation](https://docs.gradle.org/current/userguide/java_library_plugin.html#sec:java_library_separation)
- [Gradle User Guide — Declaring dependencies](https://docs.gradle.org/current/userguide/declaring_dependencies.html)
