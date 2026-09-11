---
title: Compile First, Then Run scalafixAll --check on CI
impact: HIGH
impactDescription: a scalafix violation fails the visible PR diff instead of being silently auto-fixed and merged as-is by the CI job itself
tags: [scalafix, ci, sbt-scalafix, check-mode]
---

# Compile First, Then Run scalafixAll --check on CI [HIGH]

## Description
`sbt scalafixAll` applies every configured rule's fixes in place and exits `0` regardless of whether it changed anything — the same asymmetry as running plain `scalafmt` on CI (see [`scalafmt-ci-check-mode`](scalafmt-ci-check-mode.md)). `sbt "scalafixAll --check"` instead reports what would change and exits non-zero if anything would, without touching any file, which is what turns a scalafix violation into a build failure a reviewer actually sees on the PR.

Because semantic rules need SemanticDB output from a real compile (see [`scalafix-semantic-rules-require-semanticdb`](scalafix-semantic-rules-require-semanticdb.md)), the CI job has to compile before it checks — running `scalafixAll --check` against a project that hasn't been compiled with `semanticdbEnabled` fails with a SemanticDB error that has nothing to do with an actual rule violation, which is easy to misread as "scalafix is broken" rather than "compile didn't run first."

## Bad Example
```yaml
# CI — mutates in place and always exits 0; violations merge silently
- run: sbt scalafixAll
```
```yaml
# CI — check runs before any compile; fails on missing SemanticDB, not on real violations
- run: sbt "scalafixAll --check"
```

## Good Example
```yaml
# CI
- run: sbt compile
- run: sbt "scalafixAll --check"
```

## Notes
- `scalafixAll` covers both `Compile` and `Test` sources in one invocation; `scalafix` alone targets a single configuration.
- Keep a local pre-commit or pre-push hook running `sbt scalafixAll` (write mode) so violations are fixed before CI's check mode ever sees them.
- A CI cache that restores `target/` from a previous run can carry stale SemanticDB output — invalidate it on `.scalafix.conf` or source changes, not just on dependency changes.

## References
- [Scalafix — Continuous Integration](https://scalacenter.github.io/scalafix/docs/users/installation.html#sbt)
