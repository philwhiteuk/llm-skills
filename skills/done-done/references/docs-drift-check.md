# Documentation drift-check (non-blocking, runs last)

This is **not** a merge gate — it never blocks a merge. It's a final polish pass that runs *after* the other work, because resolving comments, fixing CI, and rebasing can leave the title/description describing a PR that no longer exists. Make a best-effort fix and move on; an honest description is worth a minute, but blocking an approved, green PR over a doc technicality is exactly what tempts an `--admin` override this skill won't use.

## The three checks

Read the current state and the actual diff:

```bash
gh pr view <N> --json title,body,headRefName,commits,files
gh pr diff <N>
```

1. **Does the title/description still match the change?** If remediation altered what the PR does (you dropped a file, the conflict resolution changed behaviour, the CI fix reworked an approach), update the prose so it's accurate.
2. **Does the description capture all the behavioural changes?** Skim the diff for behavioural changes the body doesn't mention and add them. Describe behaviour, not a file list.
3. **Does the title carry a correct issue reference?** Look for the issue key in the branch name (`PROJ-123/...`, `123-...`), commit messages, or existing body. If you find one, make sure it's in the title (e.g. `PROJ-123: <summary>`). **Don't fabricate** a reference — if there's genuinely no issue anywhere, note it and move on; its absence doesn't block the merge.

## Editing the description

Preserve the author's voice — **add to or correct** the existing body, don't wholesale replace a description someone wrote. Keep it tight (a few hundred words is plenty; the goal is a future reader's understanding, not length).

```bash
gh pr edit <N> --title "<corrected title>"
gh pr edit <N> --body-file - <<'EOF'
<updated body>
EOF
```

## Report back

What drifted (title / body / issue ref), what you fixed best-effort, and anything you couldn't infer without guessing — remembering none of it gates the merge.
