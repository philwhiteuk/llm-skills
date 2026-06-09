# G2 — make the branch mergeable

Gate G2 fails when the branch can't merge as-is: `mergeable == "CONFLICTING"` (conflicts with base) or `mergeStateStatus == "BEHIND"` (out of date). For a fresh agent-drafted branch this usually passes untouched — this step is the rare branch, not the common path.

Two reasons to handle it **early**, before review:

1. CI should run against the real merge result, not a stale base.
2. Rebasing *after* an approval can trip "dismiss stale reviews" on protected branches and silently drop the approval. Rebase while the PR is still unreviewed.

## Branch policy: rebase, not merge-commit

The branch should carry only its own commits against base — a linear history. So when base moves, **rebase**; don't merge base in. Rebasing rewrites history and needs a force-push, which is allowed here **only** with `--force-with-lease` and **only** while no review exists yet. A bare `git push --force` is never acceptable.

## BEHIND — bring the branch current

```bash
gh pr checkout <N>
git fetch origin <baseRefName>
git rebase origin/<baseRefName>
git push --force-with-lease
```

A clean rebase re-triggers CI, so the next pass may show G4 pending — that's expected; don't wait on it (push-and-return).

## CONFLICTING — resolve only what's unambiguous

After the rebase (or on a plain conflict), you may hit conflicts. Resolve a conflict **only when the intended result is obvious** — e.g. both sides added an import, a lockfile regenerates deterministically, a changelog where both entries clearly belong. Then:

```bash
git add -A
git rebase --continue
git push --force-with-lease
```

If a conflict is **ambiguous** — two real changes to the same logic, where picking one could drop someone's work — do **not** guess. Abort and escalate:

```bash
git rebase --abort
```

A conflict needing a human decision is orchestrator-resolvable: report in the terminal which files/hunks conflict and why you can't safely resolve them, so the operator can take over. Don't leave the branch in a half-rebased state.

## Report back

What state the branch was in (`BEHIND`/`CONFLICTING`), whether you rebased and force-pushed-with-lease, which conflicts you resolved and how, which you escalated and why, and whether G2 now passes (noting CI will re-run if you pushed).
