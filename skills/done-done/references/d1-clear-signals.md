# D1 worker — clear "not finished" signals

You are the D1 worker. The gate fails when the PR carries author-intent signals saying the work isn't ready:

- A `wip` / `do-not-merge` / `hold` / `blocked` label
- A title prefixed `WIP`, `[WIP]`, `Draft:`, or `DRAFT`
- Unresolved review comments left by the PR author themselves (self-review feedback the author wants addressed before peer review)

These signals exist because the author put them there. Do not strip them silently — that overrides a deliberate human signal and can cause the orchestrator to merge work the author meant to hold.

## What to do

### Part A: Labels and title signals

1. Inspect the PR:

   ```bash
   gh pr view <N> --json title,labels,author
   ```

2. Decide which signal(s) are present — labels, title prefix, or both.

3. Look at the most recent commits on the branch (last 5) and the most recent comments. Is there evidence the author considers the work finished?

   ```bash
   gh pr view <N> --json commits,comments --jq '{commits: [.commits[-5:][] | {message: .messageHeadline, when: .committedDate}], comments: [.comments[-3:][] | {body, author: .author.login, when: .createdAt}]}'
   ```

   Signals that the work is finished and the WIP marker is stale: recent commit messages like "final tweaks", "ready", "done", or author comments saying so.

4. If the evidence is clear that the marker is stale:
   - Remove offending labels: `gh pr edit <N> --remove-label wip,do-not-merge,hold,blocked` (only the ones actually present).
   - Strip the title prefix: `gh pr edit <N> --title "<new-title>"` — preserve everything after the prefix.

5. If the evidence is ambiguous or the marker looks current, do **not** touch it. Report: "D1 still failing — author's WIP signal looks intentional; ask the user."

### Part B: Author self-review comments

Authors commonly draft a PR and review it themselves, leaving comments that flag things they want fixed before peer review. These are "not finished" signals — just as deliberate as a WIP label.

1. Fetch review comments on the PR:

   ```bash
   gh api repos/<owner>/<repo>/pulls/<N>/comments --jq '[.[] | select(.user.login == "<author-login>") | {id, path, body, in_reply_to_id, created_at}]'
   ```

2. Also fetch review-level (non-inline) comments from the PR author:

   ```bash
   gh api repos/<owner>/<repo>/pulls/<N>/reviews --jq '[.[] | select(.user.login == "<author-login>" and .state == "COMMENTED" and .body != "") | {id, body, submitted_at}]'
   ```

3. For each author comment, determine if it has been addressed:
   - **Has a reply from someone else (or a subsequent commit touching that file/line)** → likely addressed
   - **Standalone with no reply and no subsequent commit** → unresolved

4. For each unresolved author comment:
   - **Clear fix (typo, naming, null check, missing test case, style, TODO the author flagged)** — apply it, commit, push, then reply to the comment saying what you did.
   - **Design decision, scope question, or ambiguous** — do **not** guess. Skip it and surface it in your report.

This work runs in parallel with D2 (CI). Both must pass before the orchestrator flips D3 (ready for review).

## Report back

One short paragraph:
- Which signals were present (labels, title, self-review comments)
- Whether you cleared them and why
- How many author self-review comments were found, how many addressed, how many skipped
- Whether the gate now passes
