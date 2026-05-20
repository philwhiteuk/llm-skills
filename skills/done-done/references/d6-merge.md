# D6 — merge

The orchestrator runs this inline, not as a worker. This file documents the merge decision.

## Preconditions

Only attempt to merge when **all** of:

- D1 ✓ (no WIP signals)
- D2 ✓ (checks green)
- D3 ✓ (not draft)
- D4 ✓ (peer approved by a non-author)
- D5 ✓ (documented)
- `mergeStateStatus == CLEAN`

Any other `mergeStateStatus` value (`BEHIND`, `BLOCKED`, `DIRTY`, `UNSTABLE`, `UNKNOWN`) means GitHub itself is not ready to merge — do not try to force it.

## Action

Default to squash:

```bash
gh pr merge <N> --squash --auto
```

If the user has stated a preference (`--merge`, `--rebase`), use that instead. `--auto` lets GitHub merge as soon as required checks pass, which is harmless once everything is already green.

## After merge

GitHub will set `merged: true` and `mergedAt`. The next orchestrator tick will see D6 pass, emit the DONE block, and `/loop` will terminate on the `Verdict: DONE` token.

## Edge cases

- **`mergeStateStatus == BEHIND`**: the branch needs to be brought up to date. Run `gh pr update-branch <N>` (or merge `main` into the branch and push) before retrying. This will re-trigger CI, so the next tick may show D2 pending — that's fine.
- **`mergeStateStatus == DIRTY`**: merge conflict. Spawn a one-off conflict-resolution worker or hand back to the user — conflicts are not deterministic enough to auto-resolve in this skill.
- **Required reviews configured beyond one**: D4 only checks for one non-author approval. If branch protection requires two, the merge will fail with a clear message; surface it and the user can chase the second reviewer.
