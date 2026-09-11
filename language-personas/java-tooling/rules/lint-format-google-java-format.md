---
title: Adopt google-java-format for Zero-Config, Deterministic Formatting
impact: MEDIUM
impactDescription: eliminates whitespace/brace-placement review comments and formatting bikeshed permanently
tags: [google-java-format, formatting, code-style, tooling]
---

# Adopt google-java-format for Zero-Config, Deterministic Formatting [MEDIUM]

## Description
google-java-format reformats Java source to one of exactly two fixed styles — Google style (2-space indent) or the AOSP variant (4-space) — with essentially no further configuration surface: no brace-placement option, no configurable line-length threshold, no per-project tuning knobs. That absence of options is a deliberate design choice, not a missing feature. A formatter with configuration knobs invites a team to spend real meeting time debating whitespace preferences that produce zero functional difference in the compiled program, and a shared formatting-config file becomes one more thing that can merge-conflict and block unrelated work. A zero-config formatter converts "how should this be formatted" from an ongoing, re-litigated debate into a single one-time decision — pick Google or AOSP style — that never needs revisiting.

The practical review consequence is larger than it looks: once the formatter is wired into the build (typically through `lint-format-spotless-apply-check.md`, which handles the apply/check task split), review comments about brace placement, indentation, or spacing become structurally impossible to make, because the formatter already produced the one deterministic answer before the diff was opened. That frees review time for logic and design, which a formatter cannot judge.

## Bad Example
```java
public class OrderService
{
    public Order create( String customerId, List<Item> items ) {
        return new Order(customerId,items);
    }
}
```
Hand-formatted, IDE-default, or per-developer whitespace — every author's diffs carry incidental formatting noise alongside the actual change.

## Good Example
```java
public class OrderService {
  public Order create(String customerId, List<Item> items) {
    return new Order(customerId, items);
  }
}
```
Identical output regardless of which developer or IDE produced the change.

## Notes
- Byte-for-byte deterministic output on identical input matters beyond style consistency — it keeps `build-reproducible-builds.md` intact, since a formatter that produced different output run-to-run would itself be a source of build non-determinism.
- google-java-format reformats whitespace, line wrapping, and import grouping; it does not rename identifiers or alter semantics — it is not a refactoring tool and won't fix a bad name or a magic number.
- Running it standalone (`java -jar google-java-format.jar --replace`) works, but wiring it through Spotless is preferred so the apply/check split and CI gating come for free.

## References
- [google-java-format — GitHub](https://github.com/google/google-java-format)
- [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)
