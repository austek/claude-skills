---
title: Commit .scalafmt.conf with a Pinned version Key
impact: HIGH
impactDescription: every contributor and CI run formats with the identical scalafmt release, eliminating reformat-only diffs caused by version drift
tags: [scalafmt, configuration, reproducibility]
---

# Commit .scalafmt.conf with a Pinned version Key [HIGH]

## Description
`.scalafmt.conf` is HOCON, and `version` is the one key scalafmt refuses to run without: since scalafmt 2.x, the CLI reads `version` from the config file, downloads that exact scalafmt release (via Coursier) if it isn't already cached, and formats with it — running `scalafmt` in a repo whose config has no `version` key fails outright with an error asking for one, rather than silently falling back to whatever version happens to be installed locally.

That refusal is what makes the config portable: without a pinned version, one contributor's globally-installed scalafmt and another's IDE-bundled scalafmt (or a newer release CI happens to have cached) can disagree on formatting details across releases, so `scalafmt` reformats files differently depending on who — or what — ran it, and every such run produces a diff unrelated to the change being made. Pinning `version` guarantees the exact same formatting decisions everywhere the config is used: a contributor's terminal, their editor's format-on-save (Metals and the IntelliJ Scala plugin both read `version` from this file), and CI.

Bump `version` deliberately, in its own commit, so a formatting-only diff from the upgrade is never mixed into a change that also touches logic.

## Bad Example
```
# .scalafmt.conf — no version key
runner.dialect = scala3
maxColumn = 100
```

## Good Example
```
# .scalafmt.conf
version = "3.8.3"
runner.dialect = scala3
maxColumn = 100
```

## Notes
- `runner.dialect` should match the project's Scala version family (`scala3`, `scala213`, `scala212source3`) so scalafmt parses syntax the same way the compiler does.
- Metals and the IntelliJ Scala plugin auto-detect and use the committed `.scalafmt.conf` — no separate per-editor formatter configuration needed once this file exists.
- Pair this with [`scalafmt-ci-check-mode`](scalafmt-ci-check-mode.md) so an unformatted commit fails the build instead of merging silently.

## References
- [Scalafmt — Configuration](https://scalameta.org/scalafmt/docs/configuration.html)
