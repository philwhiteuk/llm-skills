# D6 — merge

The orchestrator runs this inline, not as a worker. This file documents the merge decision.

## Hard rules

- **Never use `--admin`.** Admin override bypasses branch protection. This skill does not bypass anything — it waits for gates to pass legitimately.
- **Never use `--auto`.** Auto-merge speculatively queues a merge for when conditions are met. This skill only merges when all conditions are met **right now**. The `/loop` wrapper handles retrying on the next tick.
- **If waiting on a review, there is nothing to do.** Do not attempt to merge, do not queue an auto-merge, do not nag reviewers. Report the status and let the next tick check again.

## Preconditions

Only attempt to merge when **all** of:

- D1 ✓ (no WIP signals, no unresolved self-review comments)
- D2 ✓ (checks green)
- D3 ✓ (not draft)
- D4 ✓ (peer approved by a non-author)
- D5 ✓ (documented)
- `mergeStateStatus == CLEAN`

If any gate is not passing, **do nothing**. Do not attempt the merge. The orchestrator will re-evaluate on the next tick.

Any `mergeStateStatus` other than `CLEAN` (`BEHIND`, `BLOCKED`, `DIRTY`, `UNSTABLE`, `UNKNOWN`) means GitHub itself is not ready to merge — do not try to force it.

## Action

Default to squash:

```bash
gh pr merge <N> --squash
```

If the user has stated a preference (`--merge`, `--rebase`), use that instead. No other flags.

## After merge

GitHub will set `merged: true` and `mergedAt`. The next orchestrator tick will see D6 pass, emit the DONE block, and `/loop` will terminate on the `Verdict: DONE` token.

## Edge cases

- **`mergeStateStatus == BEHIND`**: the branch needs to be brought up to date. Run `gh pr update-branch <N>` (or merge `main` into the branch and push) before retrying. This will re-trigger CI, so the next tick may show D2 pending — that's fine.
- **`mergeStateStatus == DIRTY`**: merge conflict. Spawn a one-off conflict-resolution worker or hand back to the user — conflicts are not deterministic enough to auto-resolve in this skill.
- **Required reviews configured beyond one**: D4 only checks for one non-author approval. If branch protection requires two, the merge will fail with a clear message; surface it and the user can chase the second reviewer.
- **Waiting on review**: If D4 is failing because no reviewer has approved yet, this is not a problem to solve — it is a state to report and wait on. There is nothing the skill can do to make a human approve faster.
