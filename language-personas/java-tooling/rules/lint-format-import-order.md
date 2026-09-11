---
title: Enforce Deterministic Import Order, Ban Wildcard Imports
impact: LOW
impactDescription: removes a recurring class of merge conflicts caused purely by import-block reordering
tags: [imports, formatting, code-style, checkstyle]
---

# Enforce Deterministic Import Order, Ban Wildcard Imports [LOW]

## Description
Import order has zero effect on compiled program behavior, but a real effect on diff noise and merge conflicts. Without an enforced order, two branches that each add a different import to the same file frequently produce a merge conflict purely from where the new line landed in an unordered block, even though neither change touches the same logical import. A single deterministic order — alphabetical within groups, typically `java.*`, then `javax.*`, then third-party, then project-internal packages, with static imports grouped separately — eliminates that entire class of conflict and makes a diff immediately legible: an added, removed, or genuinely reordered import stands out instead of blending into an arbitrarily-sorted block.

Wildcard imports (`import java.util.*;`) compound the problem in a different way: they hide exactly which types a file actually depends on, defeating the purpose of an import list as an explicit, greppable declaration of a class's dependencies. A wildcard import can also silently change a file's meaning if a new type is later added to the wildcard-imported package under a name that collides with something already in scope — a failure mode an explicit import list cannot produce, since adding a name to a package doesn't retroactively make an unrelated file import it.

Both concerns are fully mechanical, so neither belongs in review. google-java-format expands wildcards and sorts imports on reformat; Checkstyle's `ImportOrder` and `AvoidStarImport` checks enforce the same rules for teams not using that formatter.

## Bad Example
```java
import java.util.*;
import com.example.billing.Invoice;
import java.time.Instant;
```

## Good Example
```java
import java.time.Instant;
import java.util.List;
import java.util.Map;

import com.example.billing.Invoice;
```

## Notes
- Set the IDE's auto-import "use wildcard when N+ classes imported" threshold high enough to never trigger (e.g. 999 in IntelliJ) — otherwise the IDE inserts a wildcard the formatter or CI gate then rejects, creating friction on every save.
- This is enforced identically by `lint-format-google-java-format.md` (formatter-driven) or `lint-format-checkstyle-config.md` (linter-driven) — pick one enforcement point, not both, to avoid the two-source-of-truth conflict described in the Checkstyle rule.
- Static imports are conventionally grouped and ordered separately from regular imports; most style guides exempt them from the "no wildcard" rule for a small, fixed set of cases like `static org.assertj.core.api.Assertions.*` in test code, though the stricter reading forbids that too.

## References
- [Google Java Style Guide — Import Statements](https://google.github.io/styleguide/javaguide.html#s3.3-import-statements)
- [Checkstyle — ImportOrder](https://checkstyle.org/checks/imports/importorder.html)
