# claude-skills

Personal Claude Code skills and commands, packaged as a plugin marketplace.

## Install

In Claude Code:

```
/plugin marketplace add austek/claude-skills
```

Then enable whichever plugins you want:

- `language-personas` — Java/Python/Rust/Scala coding-standards, testing, and
  tooling personas, plus a JVM systems mentor and a polyglot tutor.
- `pr-workflow-oss` — open/review/land a PR on a personal or open-source repo.
- `architecture` — deep-module design, domain modeling, and architecture
  improvement skills.
- `productivity` — plan-sharpening interviews, agent handoffs, and
  questionnaire generation.
- `dev-utilities` — misc skills: IDE MCP tool selection, comment
  cleanup, code-claim verification, business analysis.

Some skills in `architecture` and `productivity` are adapted from
[mattpocock/skills](https://github.com/mattpocock/skills) (MIT) — see
`THIRD-PARTY-LICENSES.md`.

Note: `analysis`, `pr`, and `story` commands (in `dev-utilities`) read voice
and style rules from your own project's `CLAUDE.md`. Without one, they fall
back to sensible defaults.
