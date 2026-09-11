---
title: Baseline Wartremover and Scalastyle Instead of Enforcing Everywhere on Day One
impact: HIGH
impactDescription: new code gets enforcement immediately without a red, unfixable build blocking every PR against the existing codebase
tags: [wartremover, scalastyle, legacy-code, adoption, baseline]
---

# Baseline Wartremover and Scalastyle Instead of Enforcing Everywhere on Day One [HIGH]

## Description
Turning `wartremoverErrors` or a Scalastyle `error`-level check on repo-wide against an existing, unvetted codebase in one step typically produces hundreds of pre-existing violations the team didn't just introduce, and a build that's red everywhere blocks all further development until someone fixes all of them — which in practice means the tool gets disabled again rather than adopted. Both tools support scoping enforcement to a subset instead: Wartremover's `wartremoverExcluded` setting takes a list of file paths to skip entirely, and `Wart.allBut(Wart.Any, Wart.Nothing)` builds "every built-in wart except these" for teams that want broad coverage minus a few noisy ones; Scalastyle supports the same idea per-check via `enabled="false"` in the XML, or per-block with inline `// scalastyle:off <checkerId>` / `// scalastyle:on <checkerId>` comments around code that can't be brought into compliance immediately.

The adoption path that actually sticks: enforce the full wart/check set only on new or already-refactored packages, exclude the rest of the legacy tree explicitly, and shrink that exclusion list over time as modules get touched anyway — the same incremental-rollout shape as scoping a static analyzer to specific packages on an existing Java codebase.

## Bad Example
```scala
// build.sbt — flips every wart to fatal across the entire existing codebase at once
Compile / compile / wartremoverErrors ++= Wart.allBut(Wart.Any)
// hundreds of pre-existing violations; build is red everywhere, blocking unrelated PRs
```

## Good Example
```scala
// build.sbt — enforced only on the module actively being cleaned up
Compile / compile / wartremoverErrors := Seq(Wart.Null, Wart.Var)
wartremoverExcluded += baseDirectory.value / "src" / "main" / "scala" / "legacy"
```

## Notes
- Track the exclusion list itself (a comment with a ticket reference, or a dated TODO) so it's visibly shrinking rather than becoming permanent.
- The same strategy applies to compiler-flag adoption — see [`compiler-flags-xfatal-warnings`](compiler-flags-xfatal-warnings.md) for `-Wconf`-based selective fatality instead of an all-or-nothing switch.
- Scalastyle's inline `// scalastyle:off`/`// scalastyle:on` should wrap the smallest block that needs the exemption, not an entire file, so the suppression doesn't silently cover unrelated code added later in the same file.

## References
- [Wartremover — Excluding Code](https://www.wartremover.org/doc/install-setup.html#excluding-code)
- [Scalastyle — Suppressing Checks](http://www.scalastyle.org/inline.html)
