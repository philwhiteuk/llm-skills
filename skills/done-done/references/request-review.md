# G6 — get a peer review

Gate G6 fails until a non-author has approved. You can't make a human approve, but you can make sure the *right* person is asked. (Addressing in-flight comments lives in the Comments step; this step is purely reviewer selection.)

If `reviewRequests` already has reviewers and you're just waiting on their approval, there's nothing to do — that's an indeterminate human wait. Report it and return.

## Choose the reviewer deliberately

If no one is requested yet, pick the person most likely to give a fast, informed review — don't grab the first name from a list. Work down this priority order; the higher signals are more authoritative:

1. **User-named** — if the operator named a reviewer in the prompt, use that, full stop.
2. **CODEOWNERS** — if the repo has one and it covers the changed paths, that's the org's own declared answer (GitHub may auto-request on its own once the draft goes ready; confirm by re-fetching `reviewRequests`).
3. **File git-history** — who recently and frequently touched the files this PR changes:

   ```bash
   gh pr view <N> --json files --jq '.files[].path' | head -20 \
     | xargs -I{} git log --format='%an <%ae>' -5 -- {} \
     | sort | uniq -c | sort -rn | head -5
   ```

4. **Recent repo contributors** — only if the files are brand new / history is thin:

   ```bash
   git log --format='%an <%ae>' -50 | sort | uniq -c | sort -rn | head -5
   ```

**Exclusions:** never the PR author; skip bots and clearly inactive accounts.

Request 1–2 reviewers, never a crowd (spraying five people diffuses responsibility):

```bash
gh pr edit <N> --add-reviewer <login>[,<login>]
```

## When the best candidate is the operator

In the agent-drafted case the author is often a bot and the obvious human reviewer is the person running this skill. Don't quietly self-assign them — the point of this gate is an *independent* set of eyes, and routing it back to the operator blurs that. Instead surface it in the terminal: "ready for **your** review — you're the strongest candidate (file history / CODEOWNERS)." Let them decide to review or name someone else.

## When you can't tell

If none of the signals give a confident answer, don't guess. This is orchestrator-resolvable: report in the terminal and ask who should review.

## Report back

Who you requested and on which signal (user-named / CODEOWNERS / file-history / recent-contributor), or that the best candidate is the operator (surfaced, not assigned), or that you couldn't determine one — and whether the PR is now waiting on a human approval.
