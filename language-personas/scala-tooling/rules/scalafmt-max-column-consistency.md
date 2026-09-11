---
title: Pick One maxColumn Value and Keep the IDE Reading the Same Config
impact: MEDIUM
impactDescription: format-on-save in the editor and scalafmtCheckAll on CI agree on every file, instead of fighting over line width on every PR
tags: [scalafmt, maxColumn, configuration, consistency]
---

# Pick One maxColumn Value and Keep the IDE Reading the Same Config [MEDIUM]

## Description
`maxColumn` controls the line width scalafmt wraps at and defaults to 80 if unset — narrower than what most Scala teams actually use, so an explicit choice (100 and 120 are both common) belongs in the committed `.scalafmt.conf` rather than left implicit. The value matters less than everyone agreeing on the same one: scalafmt's line-wrapping decisions (where a method chain breaks, whether a parameter list goes multi-line) change at the margins between adjacent `maxColumn` values, so any tool reading a different number than the committed config reformats differently.

The practical failure mode is an editor whose Scala plugin has its own formatter setting independent of the project's `.scalafmt.conf` — IntelliJ's Scala plugin and Metals both read the committed file automatically when configured to delegate to scalafmt, but a misconfigured editor, or one falling back to a built-in formatter instead of invoking scalafmt itself, silently uses its own default. Format-on-save then produces output that disagrees with `scalafmtCheckAll` on CI, and every save from that editor produces a diff CI rejects.

## Bad Example
```
# .scalafmt.conf — maxColumn left at the implicit default of 80
version = "3.8.3"
runner.dialect = scala3
```
*(IDE's own formatter is set to 120 columns, disagreeing with the file scalafmt actually uses)*

## Good Example
```
# .scalafmt.conf — explicit, and the only line-width setting anyone needs to configure
version = "3.8.3"
runner.dialect = scala3
maxColumn = 100
```

## Notes
- Confirm the IDE is delegating to the project's scalafmt (Metals does this by default; IntelliJ's Scala plugin needs "Use scalafmt formatter" enabled) rather than using a built-in formatter with its own width setting.
- `maxColumn` changes are reformatting-only — bump it in its own commit, same discipline as a `version` bump under [`scalafmt-config-committed`](scalafmt-config-committed.md).
- Multi-module repos should share one root `.scalafmt.conf`; per-module overrides reintroduce the same inconsistency this rule exists to avoid.

## References
- [Scalafmt — Configuration Reference](https://scalameta.org/scalafmt/docs/configuration.html)
