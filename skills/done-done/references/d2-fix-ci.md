# D2 worker — fix CI failures

You are the D2 worker. The gate fails when one or more checks in `statusCheckRollup` are not in `{SUCCESS, NEUTRAL, SKIPPED}`, or the rollup is empty, or jobs are still pending. Your job is to make the checks green.

## Loop

Run this loop up to **3 cycles**, then stop and report whatever is still red.

### 1. Identify what's failing

```bash
gh pr checks <N>
```

If anything is `IN_PROGRESS` / `QUEUED` / `PENDING`, wait it out — `gh pr checks <N> --watch` is fine for short waits, but if more than ~5 minutes remain, return control and let the next `/loop` tick pick it up. There is no point burning tokens on a sleep.

For each failing check, grab the logs:

```bash
gh run view <run-id> --log-failed
```

### 2. Reproduce locally

Check out the PR branch if you're not already on it:

```bash
gh pr checkout <N>
```

Then reproduce the failure with the same command CI used. Common patterns:

- **Lint**: `npm run lint` (or the repo's lint script). Try `npm run lint -- --fix` first; fix the remainder manually.
- **Type errors**: `npx tsc --noEmit` or the repo's typecheck. Read the errors, fix the types, do not `any`-cast unless the original code already does so for the same reason.
- **Tests**: run the failing suite. Read the assertion. **Fix the code, not the test** — only update the test if the behaviour change was intentional and the test is now stale.
- **Build**: `npm run build`. Read the error, fix imports / configs / missing deps.

### 3. Push the fix

```bash
git add -A
git commit -m "fix: <one-line summary of what you fixed>"
git push
```

Never force-push. Never skip hooks. If a hook fails, fix the underlying issue and make a new commit.

### 4. Re-check

After pushing, return — do not block waiting for CI to re-run. The orchestrator's next tick will fetch fresh state and re-dispatch if needed. If you want one quick re-check before returning, `gh pr checks <N>` is fine, but do not enter a busy-wait.

## Hard limits

- **Max 3 fix-push cycles per invocation.** If still red after that, return with a report of what remains and let the user intervene.
- **Never modify test assertions** to make a test pass unless the underlying behaviour change was intentional and the test really is stale. Tests catch bugs; defeating them defeats the point.
- **Never disable a check** (skip CI, change required-status rules, edit workflows to make them skip) to bypass D2. That is cheating the gate.
- **If a failure requires a design decision** (e.g. an integration test failed because the expected behaviour is genuinely unclear), stop and ask the user via the report.

## Report back

- Which checks were failing
- What you did to fix each one (file:line of the fix, or a one-liner per check)
- Which checks are now green and which (if any) remain
- Any place you bailed out and need user input
