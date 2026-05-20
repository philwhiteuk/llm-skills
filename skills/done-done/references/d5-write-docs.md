# D5 worker — make the PR adequately documented

You are the D5 worker. The gate fails when the PR `body` is shorter than 200 non-whitespace characters or contains no recognisable issue link. Your job is to make the description meaningful to a future reader without making things up.

## The gate, restated

D5 passes when **both**:

1. `body` has ≥ 200 non-whitespace characters
2. `body` matches one of: `#\d+` (GitHub issue), `\b[A-Z][A-Z0-9]+-\d+\b` (e.g. `PROJ-123`), or `(?i)(closes|fixes|resolves)\s+\S+`

## What to do

### 1. Find the missing pieces

```bash
gh pr view <N> --json title,body,headRefName,commits,files
```

- If the body is empty or trivially short → you need to write one.
- If the body is long enough but has no issue link → you need to find and add the link.

### 2. Find the issue link (do not invent one)

Look in this order:

- The **branch name** often encodes the issue: `PROJ-123/foo-bar` or `123-add-foo`. Extract it.
- The **commit messages** on the branch — `gh pr view <N> --json commits --jq '.commits[].messageHeadline'`. Many teams put the issue key in commits.
- The **PR title** — same pattern.
- Recent comments on the PR.

If you find a plausible issue reference, add a `Closes #<n>` or `Closes <KEY>` line to the body.

If you cannot find an issue reference anywhere, **do not fabricate one**. Report back: "no issue reference found in branch / commits / title — ask the user".

### 3. Write the body (only if missing or trivially short)

Read the diff to understand what the PR actually does:

```bash
gh pr diff <N>
```

Write a body with this shape:

```
Closes <ISSUE>

## What
<one-paragraph summary of what changed, in the user's voice not the AI's — describe the
behavioural change, not the file list>

## Why
<one paragraph explaining the motivation — the problem this solves, the constraint it
unblocks, or the bug it fixes>

## Notes for reviewers
<optional — anything subtle, anything to skip, anything the reviewer should know up front>
```

Keep the whole thing under ~400 words. The 200-char floor is a sanity check, not a target — verbose descriptions are not better than concise ones.

Update the body:

```bash
gh pr edit <N> --body-file - <<'EOF'
<your body>
EOF
```

### 4. Never overwrite an existing body

If the author already wrote a body and it's just under the 200-char floor or missing the issue link, **add** to it — don't replace it. Preserve the author's voice and prepend or append your additions.

## Report back

- What the body was before (length, had-link y/n)
- What you added (issue link, prose, both)
- Whether the gate now passes
- Anything you needed to skip because you couldn't infer it without guessing
