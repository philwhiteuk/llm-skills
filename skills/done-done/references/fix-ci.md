# G4 — fix CI failures

Gate G4 fails when a check isn't in `{SUCCESS, NEUTRAL, SKIPPED}`, the rollup is empty, or jobs are still pending. In the common case these are the lint/type/test rules the drafting agent didn't know about — usually quick, mechanical fixes.

## The loop boundary

**Loop on local work; don't wait on remote CI.** Reproduce the failure *locally*, fix it, and re-run locally as many times as it takes — that's synchronous work you control. Once you push, remote CI is asynchronous and may take minutes; you can't confirm the outcome from inside this pass, so **push and return**, reporting that CI is re-running and unverified. Don't busy-wait or sleep.

## 1. See what's failing

```bash
gh pr checks <N>
```

If everything is merely `IN_PROGRESS`/`QUEUED`/`PENDING`, there's nothing to fix yet — report "CI in progress, unverified" and return; a later run will pick up the result. For genuinely failed checks, pull the logs:

```bash
gh run view <run-id> --log-failed
```

## 2. Reproduce locally and fix

```bash
gh pr checkout <N>
```

Run the same thing CI runs:

- **Lint** — `npm run lint` (or the repo's script). Try `--fix` first, then clean up the rest by hand.
- **Types** — `npx tsc --noEmit` (or the repo's typecheck). Fix the types; don't `any`-cast unless the surrounding code already does for the same reason.
- **Tests** — run the failing suite, read the assertion. **Fix the code, not the test.** Only change a test if the behaviour change was intentional and the test is genuinely stale — never to silence a real failure.
- **Build** — `npm run build`; fix the imports/config/missing deps it points at.

## 3. Push

```bash
git add -A && git commit -m "fix: <what you fixed>" && git push
```

Never force-push here (this isn't a rebase), never skip hooks. If a hook fails, fix the cause and commit again.

## 4. Return

After pushing, return — don't block on the re-run. Report what you fixed and that CI is re-running.

## When to escalate instead of fix

If a failure is **out of scope** for this PR — a flaky/unrelated suite, an infra/credentials problem, a check failing for reasons the PR didn't cause, or a failure that needs a genuine design decision — don't paper over it and don't disable the check. This is orchestrator-resolvable: report in the terminal which check, why it's out of scope, and what you think needs to happen, so the operator can decide.

## Report back

Which checks were failing, what you changed for each (file:line or a one-liner), what you pushed, anything you escalated as out-of-scope, and the honest status: green locally / pushed and re-running / blocked.
