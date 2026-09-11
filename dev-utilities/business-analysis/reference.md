# Mechanics for `business-analysis`

Repo: `collibra/dgc-core` (not `dgc`, avoids 307 redirect). Jira facts: `../collibra-jira-ticket/reference.md`.

## 1. Discovery

```bash
gh pr list --repo collibra/dgc-core --state all --search "<KEY>" --json number,title,url,state,headRefName
# Fallback
git branch -a | grep -i <number>
gh pr view <N> --repo collibra/dgc-core --json title,body,files,additions,deletions,mergedAt,author
```

Fetch ticket (`getJiraIssue`). Read Background, Scope, parent, issuelinks. Use `jq` if fields spill to file.

## 2. Tracing

**Post-fix:** Read diff directly: `gh pr diff <N> --repo collibra/dgc-core -- <path>`

**Pre-fix:**
- Derive search terms from symptoms. Read epic/sibling links for component hints.
- Fallback search (avoids rtk issues): `grep -rliE "term" --include='*.java' --exclude-dir=build <modules>`
- Check gates: `grep -rn "ConditionalOnProperty" ...`
- Verify CI gap: search tests in `src/integrationTest` or `src/test`.

## 3. Posting to Jira

**Comments** (`addCommentToJiraIssue`): Use real Markdown. Post-fix: link PR. Pre-fix: anchor on ONE `File.java:NNN` and name commit read. Recommend field changes (Severity/Priority) in text.

**Criteria Fields:** Require ADF (`contentFormat=adf`). Check current contents before overwrite! Use `editJiraIssue`.

**ADF Payloads:**

Technical AC (`customfield_12100`):

```json
{"customfield_12100": {"type": "doc", "version": 1, "content": [{"type": "bulletList", "content": [{"type": "listItem", "content": [{"type": "paragraph", "content": [{"type": "text", "text": "<assertion>"}]}]}]}]}}
```

BDD (`customfield_16317`):

```json
{"customfield_16317": {"type": "doc", "version": 1, "content": [{"type": "codeBlock", "attrs": {"language": "gherkin"}, "content": [{"type": "text", "text": "Feature: ..."}]}, {"type": "paragraph", "content": [{"type": "text", "text": "Tester note: ", "marks": [{"type": "strong"}]}, {"type": "text", "text": "<limits>"}]}]}}
```

Bug Fallback (BDD appended to `customfield_12100`):

```json
{"customfield_12100": {"type": "doc", "version": 1, "content": [{"type": "heading", "attrs": {"level": 3}, "content": [{"type": "text", "text": "Technical (Engineering)"}]}, {"type": "bulletList", "content": ["..."]}, {"type": "heading", "attrs": {"level": 3}, "content": [{"type": "text", "text": "Business / Tester (BDD)"}]}, {"type": "codeBlock", "attrs": {"language": "gherkin"}, "content": [{"type": "text", "text": "Feature: ..."}]}]}}
```
