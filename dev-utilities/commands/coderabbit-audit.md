---
description: Run a CodeRabbit audit scoped to uncommitted changes, a path, a PR, a branch, or the full repo, and summarize the findings
argument-hint: "[path|branch|#PR|full repo] (optional; defaults to uncommitted changes)"
allowed-tools: Bash, Read
---

Run a CodeRabbit review via the `coderabbit` CLI (and, for a full-repo audit, the `coderabbitFullAudit` script installed as part of the `local` dotfiles package) and report the results.

### Step 1: Preconditions
1. Confirm the `coderabbit` CLI is on `PATH` (`command -v coderabbit`) and authenticated (`coderabbit auth status` or equivalent). If not installed, tell the user to run the `personal` profile's `setup.sh`, which installs it; if not authenticated, tell them to run `coderabbit auth login`.
2. Only for the full-repo and path scopes (see below) — both use the scoped worktree procedure: confirm the current directory is a git repository and `.coderabbit.yaml` is tracked on the current branch. If either is missing, stop and tell the user what's missing — `coderabbitFullAudit` itself also checks this and will exit with a clear error.

### Step 2: Resolve scope from `$ARGUMENTS`
Resolve `$ARGUMENTS` to a target in this order — stop at the first match:

1. **No argument** → run `git status --porcelain`. If it's empty, tell the user there are no uncommitted changes to audit and stop — do not fall back to a full-repo audit. Otherwise scope to uncommitted changes.
2. **A filesystem path** (file or directory that exists on disk) → scope to that path.
3. **A PR reference** (a number, or a `github.com/.../pull/N` URL) → scope to that pull request.
4. **A branch name** (matches an existing local or remote ref) → scope to that branch's diff.
5. **`full repo` / `repo`** → the entire repo is in scope.

### Step 3: Run the audit

- **Uncommitted changes**: `coderabbit review --uncommitted --agent`.
- **A path**: a full-content audit of just that path — not a diff — using the scoped worktree procedure below with `SCOPE_PATH="<path>"`.
- **A PR reference**: try `coderabbit pullrequest <number-or-url> --agent` first — this reads CodeRabbit's existing review of that PR (fast, requires the repo to be installed in the org and the PR already reviewed by the GitHub App). If it reports no existing review, fall back to a live review of that PR's diff: resolve its branch with `gh pr view <number-or-url> --json headRefName,baseRefName`, check out `headRefName`, and run `coderabbit review --base <baseRefName> --agent`.
- **A branch name**: `coderabbit review --base <branch> --agent`, reviewing the current checkout's diff against that branch.
- **Full repo**: run `coderabbitFullAudit` via Bash from the repo root — the packaged version of the same scoped worktree procedure below with `SCOPE_PATH="."`. This is the only scope that can take several minutes on a large repo — do not cancel early.

For every scope, redirect/export the raw review output to `~/.claude/scratches/handoff/coderabbit_audit.md` (`coderabbitFullAudit` does this itself for the full-repo case; for the other scopes, redirect the command's own output there).

#### The scoped worktree procedure (used for "a path")

`coderabbitFullAudit` has no path argument, so a path scope reproduces its technique manually, restricted to `SCOPE_PATH` instead of the whole tree. A path is still a full-content audit (like the full-repo case), not a diff, so it surfaces pre-existing issues in untouched code too — the opposite trade-off from the uncommitted/PR/branch scopes above.

1. `CURRENT_BRANCH="$(git rev-parse --abbrev-ref HEAD)"`.
2. Derive a slug from the path (e.g. `addon` → `addon`, `addon/globalPlugins` → `addon-globalplugins`) and use it to name a scratch worktree (`../cr-audit-scratch-<slug>`) and two branches (`audit-base-<slug>`, `audit-code-<slug>`) — distinct names so this can't collide with a concurrent full-repo audit or another path-scoped run.
3. `git worktree add --detach <worktree-dir>`, then `cd` into it.
4. Create an empty baseline: `git checkout --orphan audit-base-<slug>`, `git rm -rf .`, `git commit --allow-empty -m "empty initial commit"`.
5. `git checkout -b audit-code-<slug>`, then `git checkout "$CURRENT_BRANCH" -- <SCOPE_PATH> .coderabbit.yaml` (include `.coderabbit.yaml` even when it's outside `SCOPE_PATH`, so its `path_filters`/`path_instructions` still apply).
6. Strip `.coderabbit.yaml`'s negated (`!`-prefixed) `path_filters` patterns out of the snapshot with `git rm -r -f -q -- ":(glob)<pattern>"` for each — same reasoning as `coderabbitFullAudit`: those patterns govern what CodeRabbit *reports on*, not what it *uploads*, so a large vendored/binary file left in the tree can still drop the review's connection.
7. `git add .`, `git commit -m "scoped <SCOPE_PATH> snapshot for audit"`.
8. `cr review --base audit-base-<slug> --agent > ~/.claude/scratches/handoff/coderabbit_audit.md`.
9. Clean up unconditionally (even on failure): `cd` back to the original repo, `git worktree remove --force <worktree-dir>`, `git branch -D audit-base-<slug> audit-code-<slug>`, `git worktree prune`.
10. If step 6's filtering still leaves over 150 files, CodeRabbit will reject the review with a `too_many_files` error listing directory-sized candidates — report that to the user and suggest narrowing to one of them, or falling back to a live diff with `coderabbit review --dir <path> --agent` (git changes only, no pre-existing-code coverage).

Write steps 1–9 to a script file (e.g. in the session scratchpad) and run it with `bash <script>`, rather than inline in a single Bash call — a multi-line inline command containing `git rm -rf .` next to an unquoted worktree-path variable can trip a dangerous-rm safety hook that can't statically verify the variable is non-empty. Running from a script file avoids that.

### Step 4: Summarize
Read `~/.claude/scratches/handoff/coderabbit_audit.md` and present the findings grouped by severity (blocker/critical first), each with the file:line it applies to and a one-line fix suggestion. Do not restate findings CodeRabbit marked as nitpicks unless the user asks for them.
