---
title: Run Scalafmt in Check Mode on CI, Never in Write Mode
impact: HIGH
impactDescription: an unformatted commit fails the build immediately instead of merging and drifting the codebase's style over time
tags: [scalafmt, ci, sbt-scalafmt, check-mode]
---

# Run Scalafmt in Check Mode on CI, Never in Write Mode [HIGH]

## Description
Plain `scalafmt` (or the sbt-scalafmt plugin's `scalafmtAll`) rewrites files in place and exits `0` whether or not anything changed — it's a formatter, not a linter, so running it on CI as-is never fails the build even when a contributor forgot to format before pushing. `scalafmt --test` (CLI) or `scalafmtCheckAll` plus `scalafmtSbtCheck` (sbt-scalafmt, covering source files and `build.sbt`/`project/*.scala` respectively) instead compare each file against what scalafmt would produce and exit non-zero — without writing anything — the moment any file disagrees.

That distinction is the whole point of running it on CI at all: a write-mode run "fixes" the formatting as a side effect of the CI job, so the PR's diff in GitHub still shows unformatted code, reviewers approve what they see, and the actual repository state (post-CI-write) never gets pushed back to reconcile the two. Check mode instead makes an unformatted PR fail visibly, forcing the fix to happen where the diff is reviewed.

## Bad Example
```yaml
# CI job — reformats but never fails; unformatted code merges silently
- run: sbt scalafmtAll
```

## Good Example
```yaml
# CI job — fails the build if anything would change
- run: sbt scalafmtCheckAll scalafmtSbtCheck
```
```bash
# CLI equivalent, outside sbt
scalafmt --test
```

## Notes
- `scalafmtCheckAll` covers `Compile` and `Test` sources; `scalafmtSbtCheck` separately covers `build.sbt` and `project/*.scala`, which use their own scalafmt config scope.
- Requires the committed, version-pinned config from [`scalafmt-config-committed`](scalafmt-config-committed.md) — check mode against a config with no pinned `version` fails for that reason before it even compares formatting.
- Keep a local pre-commit hook running plain `scalafmt` (write mode) so contributors fix formatting before CI ever sees the diff, rather than treating CI as the first formatting pass.

## References
- [Scalafmt — CI Integration](https://scalameta.org/scalafmt/docs/installation.html#task-vs-command)
