---
title: Never Split One Package Across Multiple Modules
impact: HIGH
impactDescription: avoids a module resolution failure that only surfaces at startup, on the module path
tags: [modules, jpms, build-configuration]
---

# Never Split One Package Across Multiple Modules [HIGH]

## Description
JPMS forbids two modules on the module path from defining classes in the same package — a "split package." Unlike the classpath, which silently picks whichever class it finds first on a duplicate and leaves the rest as dead weight, the module system treats a split package as a hard resolution error at startup: `java.lang.module.FindException: Module X and Y export package Z to module...`. This most often happens by accident — a library shaded or repackaged under an existing group ID, a multi-artifact project where two Maven/Gradle modules were carelessly given overlapping package prefixes, or a fork that duplicates a package instead of extending it — and by the time it's caught, it's a startup failure in an environment the developer doesn't control, not a compile error caught locally.

The fix is structural: every package belongs to exactly one module. When two components genuinely need to share a package's classes, extract that package into its own module that both depend on, rather than letting both declare it.

## Bad Example
```
billing-core.jar   → com.example.billing.model.Invoice
billing-legacy.jar → com.example.billing.model.LegacyInvoiceAdapter
```
Both jars declare classes under `com.example.billing.model` and are placed on the module path together — this fails at module resolution, before any of the application's own code runs.

## Good Example
```
billing-model.jar  → com.example.billing.model.Invoice (owns the package)
billing-core.jar    (requires billing-model, adds its own com.example.billing.core package)
billing-legacy.jar  (requires billing-model, adds com.example.billing.legacy instead of reusing .model)
```

## Notes
- The classpath (not the module path) still tolerates split packages by silently shadowing — that leniency is exactly what makes the bug invisible until the same artifacts are placed on the module path in a production or CI environment that enforces JPMS.
- `jdeps` and `jlink --module-path` both report split-package conflicts before deployment — running them in CI catches this class of error before it reaches a runtime that resolves modules strictly.
- A build reorganization that accidentally produces a split package is exactly the case `modules-api-vs-implementation-gradle-config` guards against at the build-topology level — getting module boundaries right in the build avoids most split-package situations before they can occur.

## References
- [JEP 261: Module System — Split packages](https://openjdk.org/jeps/261)
- [Java Platform Module System — Oracle documentation](https://docs.oracle.com/javase/9/docs/api/java/lang/module/package-summary.html)
