# G7 — merge

The orchestrator runs this inline (one command), not as a sub-agent. This file documents the decision.

## Merge integrity — the lines you don't cross

- **Never `--admin`.** Admin override bypasses branch protection. This skill merges legitimately or not at all.
- **Never `--auto`.** Auto-merge speculatively queues a merge for when conditions are met. This skill merges only when every gate genuinely passes *right now*; re-running (or `/loop`) handles the retry.
- **If you're waiting on a review, there is nothing to do here.** Don't merge, don't queue auto-merge, don't nag. Report and let the next pass re-check.

## Preconditions — all of these

- G1 ✓ intent clear
- G2 ✓ mergeable
- G3 ✓ comments resolved
- G4 ✓ CI green
- G5 ✓ ready (not draft)
- G6 ✓ approved by a non-author
- `mergeStateStatus == CLEAN`

If any of these is not satisfied, **do nothing** — don't attempt the merge. Any `mergeStateStatus` other than `CLEAN` (`BEHIND`, `BLOCKED`, `DIRTY`, `UNSTABLE`, `UNKNOWN`) means GitHub itself isn't ready; the earlier gates (G2 for `BEHIND`/`DIRTY`, G4 for `UNSTABLE`, G6 for `BLOCKED`) are what bring it to `CLEAN`.

## Action

Default to squash:

```bash
gh pr merge <N> --squash
```

Use the user's stated preference if they gave one (`--merge`, `--rebase`). No other flags.

## After merge

GitHub sets `merged: true` and `mergedAt`. The next pass sees G7 pass, emits `Verdict: MERGED`, and `/loop` (if used) terminates on that token.

## Edge cases

- **Branch protection needs more than one approval.** G6 only checks for one non-author approval; if the repo requires two, the merge will fail with a clear message. Surface it — the operator can chase the second reviewer.
- **`mergeStateStatus` flips to non-`CLEAN` between assessment and merge** (someone pushed to base). Don't force it — re-assess on the next pass; G2 will bring it current again.
