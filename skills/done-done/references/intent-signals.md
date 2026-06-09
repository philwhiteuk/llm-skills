# G1 — clear stale "not finished" signals

Gate G1 fails when the PR carries an author-intent signal saying the work isn't ready:

- A `wip` / `do-not-merge` / `hold` / `blocked` label
- A title prefixed `WIP`, `[WIP]`, `Draft:`, or `DRAFT`

These signals exist because someone put them there deliberately. This step is a **kill-switch**: if the signal is intentional, the author is holding the PR and nothing else you'd do matters — stop and report. Only clear a signal you're confident is *stale*. Stripping a live signal can cause a merge the author meant to prevent.

(Author self-review *comments* are handled by the Comments step, not here — they're just comments like any other.)

## What to do

1. Inspect the signals and recent activity in one go:

   ```bash
   gh pr view <N> --json title,labels,author,commits,comments --jq '{
     title, labels: [.labels[].name], author: .author.login,
     recent_commits: [.commits[-5:][] | {msg: .messageHeadline, when: .committedDate}],
     recent_comments: [.comments[-3:][] | {body, who: .author.login, when: .createdAt}]
   }'
   ```

2. Decide whether the marker is **stale** or **live**. Evidence it's stale: recent commit messages like "final", "ready", "done", or an author comment saying the work is finished. Evidence it's live: a recent commit or comment that reasserts the hold ("still WIP", "waiting on X").

3. If clearly stale, clear only what's actually present:

   ```bash
   gh pr edit <N> --remove-label wip,do-not-merge,hold,blocked   # only the labels that exist
   gh pr edit <N> --title "<title with the prefix stripped, rest preserved>"
   ```

4. If ambiguous or live, **don't touch it**. This is orchestrator-resolvable: report in the terminal — "G1 still failing: the WIP signal looks intentional, confirm before I proceed" — so the operator decides.

## Report back

One short paragraph: which signals were present, whether you judged them stale or live, what you cleared, and whether G1 now passes.
