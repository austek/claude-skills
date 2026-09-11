---
title: Run Error Prone as a javac Plugin, Not a Separate Analysis Pass
impact: HIGH
impactDescription: catches bug patterns as compiler errors instead of runtime failures discovered later
tags: [error-prone, static-analysis, compiler, javac]
---

# Run Error Prone as a javac Plugin, Not a Separate Analysis Pass [HIGH]

## Description
Error Prone (Google) plugs directly into javac's compilation pipeline and inspects the compiler's own AST as part of every `compileJava` run, rather than re-parsing source in a separate CI task the way Checkstyle or PMD do. A check that fires at ERROR severity fails the compile itself, in the same place a syntax error would — no separate report to remember to check, no separate task to wire into `check`, and the feedback lands the moment a developer compiles locally rather than on the next CI run.

Checks ship categorized as ERROR, WARNING, or SUGGESTION, and most default to WARNING so enabling Error Prone doesn't immediately break an existing build; a team ratchets up strictness by promoting specific checks with `-Xep:CheckName:ERROR` once the codebase is clean for that check. Classic catches include a class overriding `equals` without `hashCode` (or the reverse), a `String.format`-style call whose format specifiers don't match its arguments, an ignored return value from a method annotated `@CheckReturnValue`, and reference equality (`==`) on boxed types that happens to work in tests because of integer caching and then breaks in production with a different value range.

Because these bugs are caught at the exact point they're introduced rather than surfacing later as an intermittent `HashMap` lookup failure or a subtle formatting bug in a log line, the fix is cheap — a compiler error with a specific location, not a bisected regression.

## Bad Example
```java
public final class OrderId {
    private final String value;
    public OrderId(String value) { this.value = value; }

    @Override
    public boolean equals(Object o) {
        return o instanceof OrderId other && value.equals(other.value);
    }
    // no hashCode() override — compiles fine without Error Prone, breaks HashMap/HashSet usage
}
```

## Good Example
```kotlin
// build.gradle.kts
plugins {
    id("net.ltgt.errorprone") version "4.0.1"
}
dependencies {
    errorprone("com.google.errorprone:error_prone_core:2.28.0")
}
tasks.withType<JavaCompile>().configureEach {
    options.errorprone.error("EqualsHashCode") // promotes a default-WARNING check
}
```

## Notes
- On JDK 16+, Error Prone needs `--add-exports` compiler flags to access internal javac APIs; the Gradle (`net.ltgt.errorprone`) and Maven plugins add these automatically, but a hand-invoked `javac -Xplugin:ErrorProne` needs them supplied manually.
- NullAway (see `static-analysis-nullaway.md`) ships as an Error Prone check, not a standalone tool — Error Prone must already be wired in before NullAway can run.
- `-Xep:CheckName:OFF` disables a specific check per-module when it produces false positives on a particular codebase pattern; prefer this over disabling Error Prone entirely.

## References
- [Error Prone](https://errorprone.info/)
- [Error Prone — Bug Patterns](https://errorprone.info/bugpatterns)
