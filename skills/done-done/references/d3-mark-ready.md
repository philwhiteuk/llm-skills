# D3 worker — mark ready for review

The orchestrator handles this inline, not via a worker. This file exists as a reference for what "ready for review" means and the small set of safety checks before flipping the bit.

## Preconditions

Only mark a draft PR as ready when **both** are true:

- D1 ✓ — no `wip` / `hold` / `do-not-merge` / `blocked` labels, no `WIP`/`Draft` title prefix
- D2 ✓ — checks are green

Flipping a red draft to ready for review wastes reviewers' time. Don't do it.

## Action

```bash
gh pr ready <N>
```

That's it. One command.

## Edge cases

- **Author preference.** Some authors want to hold a PR in draft for reasons that aren't captured in labels (e.g. waiting on a dependency in another repo). If the most recent author comment says anything like "hold off", "don't review yet", "waiting on X", do **not** mark it ready — ask the user instead.
- **Re-drafted.** If a PR was ready, then converted back to draft, treat that as an intentional signal from the author. Same rule: ask the user.
