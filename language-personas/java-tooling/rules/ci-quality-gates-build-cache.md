---
title: Enable a Shared Remote Build Cache So CI Time Tracks What Changed
impact: MEDIUM
impactDescription: an unrelated module's tasks hit cache instead of rerunning on every CI build
tags: [gradle, build-cache, ci, build-performance]
---

# Enable a Shared Remote Build Cache So CI Time Tracks What Changed [MEDIUM]

## Description
Gradle's build cache stores task outputs keyed by a hash of that task's declared inputs — source files, compiler arguments, dependency versions, plugin versions. When a task's input hash already exists in the cache, because another branch, another CI agent, or another developer's machine already produced that exact output, Gradle copies the cached result instead of re-executing the task. On a large multi-module build, this turns CI time into roughly a function of what actually changed in a given commit rather than the size of the whole build graph — modules untouched by a change hit cache on their compile, test, and lint tasks instead of re-running them from scratch.

A local-only cache (`~/.gradle/caches/build-cache-1`) helps a single developer's repeated local builds, but a *remote* cache shared across CI agents is what makes this compound at scale: agent A building branch X populates cache entries that agent B, building an unrelated branch Y sharing upstream modules with X, can pull without ever running those tasks itself. This effect stacks with the compile-avoidance behavior described in `build-api-vs-implementation-scope.md` — a module with a tightly-scoped `api` surface both avoids triggering unnecessary downstream recompilation and, when a task does need to run, is more likely to already have a cache hit from another agent's identical build.

## Bad Example
```properties
# gradle.properties — caching never enabled
org.gradle.caching=false
```
Every CI run recompiles and retests every module regardless of what changed, even for a change confined to one leaf module.

## Good Example
```kotlin
// settings.gradle.kts
buildCache {
    local { isEnabled = true }
    remote<HttpBuildCache> {
        url = uri("https://cache.example.internal/cache/")
        isPush = System.getenv("CI") != null // only CI populates the shared cache
    }
}
```

## Notes
- A cache hit is only as trustworthy as the task's declared inputs/outputs — a custom task with an under-declared `@Input` can return a stale cached result that's actually wrong for the current input, which is a correctness bug, not just a missed performance win.
- Tasks that are inherently non-deterministic — an archive without `reproducibleFileOrder` (see `build-reproducible-builds.md`) — don't benefit from caching and shouldn't be forced into it; fix the non-determinism first.
- `org.gradle.caching=true` in `gradle.properties` or the `--build-cache` CLI flag enables the feature; local caching alone still pays off without any remote cache configured, just with a smaller blast radius of reuse.

## References
- [Gradle User Guide — Build Cache](https://docs.gradle.org/current/userguide/build_cache.html)
- [Gradle User Guide — Configuring the Build Cache](https://docs.gradle.org/current/userguide/build_cache_use_cases.html)
