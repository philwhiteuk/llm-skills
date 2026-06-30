---
name: done-done
description: >
  Drive a pull request all the way to merged — not just check whether it is. Use this skill
  whenever the user wants a PR finished, says "get this PR done", "babysit this until it
  merges", "see this through", "make sure PROJ-123 ships", "push this over the line", or
  shares a PR they want completed. Built for the common case of an agent-drafted PR that
  needs the last mile: resolve review comments, fix the lint/type/test failures the drafting
  agent missed, rebase if it won't merge cleanly, mark it ready, request the right reviewer,
  keep the description honest, and merge once it's approved and green. It reads the PR state
  deterministically to decide what's blocking, then does every mechanical fix it safely can,
  escalating only genuine judgment calls. Never uses admin override or auto-merge. Automates
  as much as possible unattended; pair with `/loop` to keep driving across CI runs and review
  waits.
---

# Done-Done

You drive a pull request to **merged** — you don't just report whether it is. The user handed you a PR (very often one an agent drafted) and wants it finished: comments resolved, checks green, reviewed, and landed, without anyone babysitting it.

Your job each invocation: **read the PR, work down every blocker you can mechanically clear, and either merge it or return an honest account of exactly what's left and who needs to act.** "Done" is not "pushed to a branch." Done means it merged — or it can't yet, for a reason a specific human must resolve.

You operate in two layers, and keeping them separate is what makes the skill trustworthy:

- **Assessment is deterministic.** A fixed set of gates, each a pure boolean over one GitHub payload, tells you *what is blocking merge right now*. The same PR state always yields the same diagnosis — the status never flaps between runs, and a re-run keys off it reliably.
- **Action is judgment-driven.** For each failing gate you attempt the real fix — and real fixes need judgment (is this conflict safe to resolve? is this CI failure in scope? who should review?). Where judgment runs out, you escalate rather than guess.

## Merge integrity — non-negotiable

The whole point is to merge *legitimately*, never to bypass the org's rules:

- **Never** `gh pr merge --admin` or any flag that bypasses branch protection.
- **Never** `gh pr merge --auto`. You merge only when every gate genuinely passes *right now* — not speculatively.
- **Never** disable, skip, or edit a check/workflow to make a gate pass. Defeating the safety net defeats the point.
- **Force-push only with `--force-with-lease`, and only before a review exists** (e.g. rebasing a fresh branch). A bare `git push --force` is never acceptable, and rewriting history after someone has reviewed can silently dismiss their approval.

## What "finishing" looks like

For the common case — a fresh, agent-drafted branch — the path is usually: resolve a few review comments → fix the lint/type errors the drafting agent didn't know about → mark ready → request a reviewer → (wait for a human) → merge. Conflicts and rebases are the rare branch, not the spine. Optimise for the common path; handle the rest when it shows up.

---

## Inputs

Resolve the target PR in this order; stop at the first hit:

1. **Explicit argument** — a PR URL, `owner/repo#N`, or PR number the user passed.
2. **Current branch** — `gh pr view` against the working directory.
3. **Ask** — if neither resolves, ask for a PR reference and stop.

Extract `owner`, `repo`, `number` and use them for every call.

---

## Step 1 — Read the state in one pass

Token cost is dominated by the data fetch, so make essentially **one** `gh pr view` call requesting every field the gates need:

```bash
gh pr view <PR_REF> --json \
  number,title,url,isDraft,state,mergeable,mergeStateStatus,merged,mergedAt,\
  author,labels,body,headRefName,baseRefName,\
  statusCheckRollup,reviewDecision,reviews,latestReviews,reviewRequests,\
  comments,files
```

Unresolved-thread state isn't in that payload — the REST/`--json` view doesn't expose whether a review thread is resolved. When you need it (the Comments gate), fetch it with GraphQL:

```bash
gh api graphql -f query='
  query($owner:String!,$repo:String!,$num:Int!){
    repository(owner:$owner,name:$repo){
      pullRequest(number:$num){
        reviewThreads(first:100){ nodes{ isResolved isOutdated
          comments(first:1){ nodes{ author{login} body path } } } }
      }
    }
  }' -F owner=<owner> -F repo=<repo> -F num=<N>
```

If the primary fetch fails, report the failure on one line and stop. Don't retry inside the skill — re-running (or `/loop`) handles that.

---

## Step 2 — Assess the gates (deterministic)

Evaluate **every** gate as a pure function of the payload. Don't short-circuit — the user needs the whole picture. These are merge-*blocking*; Documentation is assessed separately because it never blocks a merge (see Step 4).

| ID | Gate | Passes when |
|----|------|-------------|
| **G1** | Intent clear | `labels` contains none of `wip`, `do-not-merge`, `hold`, `blocked` (case-insensitive substring) **AND** `title` doesn't start with `WIP`, `[WIP]`, `Draft:`, or `DRAFT` |
| **G2** | Mergeable | `mergeable != "CONFLICTING"` **AND** `mergeStateStatus != "BEHIND"` — i.e. no conflicts and not out of date with base |
| **G3** | Comments resolved | No `reviewThreads` node with `isResolved == false` |
| **G4** | CI green | Every `statusCheckRollup` entry's `conclusion` ∈ `{SUCCESS, NEUTRAL, SKIPPED}` **AND** none have `status` ∈ `{IN_PROGRESS, QUEUED, PENDING}`. Empty rollup → **fail** |
| **G5** | Ready | `isDraft == false` |
| **G6** | Approved | `reviewDecision == "APPROVED"` **AND** `latestReviews` has at least one `APPROVED` whose `author.login != PR.author.login` |
| **G7** | Merged | `merged == true` |

Why these, and the borderline calls codified so the answer is the same every run:

- **G2 captures only the two states you can mechanically fix** — conflicts (`CONFLICTING`) and behind-base (`BEHIND`). `BLOCKED`/`UNSTABLE` are downstream of other gates (review, CI), not independent blockers.
- **G3 treats any unresolved thread as blocking** — *whoever* left it. A thread that needs a human decision genuinely isn't ready to merge over; the action layer either resolves it or escalates to its owner.
- **G4 empty rollup fails** — no CI is not green CI. `NEUTRAL`/`SKIPPED` count as success (GitHub uses them for conditional/path-filtered jobs).
- **G6 needs both the decision and a non-author approver** — guards against self-approval via bots or co-authored commits.
- **G7 is terminal.** Once merged, every other gate is moot — emit the DONE block (Step 5) and stop.

If `G7` passes → emit DONE and stop.

---

## Step 3 — Work the blockers (judgment-driven)

The gates form a dependency graph — you can't hand a red draft to reviewers, can't merge without an approval, and rebasing late can dismiss a fresh approval. Respect this order, and **run only the steps whose gate is failing** (each is condition-gated; on a clean fresh PR most are no-ops):

```
G1 intent ─→ G2 mergeable ─→ G3 comments ─→ G4 CI ─→ G5 ready ─→ G6 review ─→ docs ─→ G7 merge
```

You are the orchestrator and you own this control flow. For each failing gate, read its playbook in `references/` and do the work. **You decide whether to do a step inline or hand it to a sub-agent** — delegate the genuinely heavy, self-contained jobs (a gnarly multi-file CI fix, a thread of review comments) to a `general-purpose` sub-agent so your own context stays clear; do the quick one-command transitions yourself. Delegation is a tool, not the structure.

| Order | Failing gate | Do this | Playbook |
|-------|--------------|---------|----------|
| 1 | **G1** intent | If a WIP/hold signal is *intentional* → stop and report (kill-switch; nothing else matters). If it's *stale* → clear it. | `references/intent-signals.md` |
| 2 | **G2** mergeable | Only if not mergeable: rebase onto base (`--force-with-lease`) and/or resolve conflicts. Do this **early** — before review — so CI runs against the real merge result and a later rebase can't dismiss an approval. Ambiguous conflict → escalate. | `references/mergeability.md` |
| 3 | **G3** comments | Resolve every unresolved thread regardless of author. Clear fix → action + reply + mark the thread resolved. Needs a decision → reply to the commenter and leave it open. Re-request review after addressing change-requests. | `references/resolve-comments.md` |
| 4 | **G4** CI | Reproduce locally, fix, push. Out-of-scope failure → escalate. | `references/fix-ci.md` |
| 5 | **G5** ready | Flip with `gh pr ready <N>` once G1 ✓, G3 ✓, and CI is green or (after your own fix) re-running — not while it's failing-and-unfixed. | inline |
| 6 | **G6** review | If no reviewer is requested, select and request one. If approved, nothing to do. Waiting on a human → return. | `references/request-review.md` |
| — | **Docs** | Non-blocking drift-check, run last (Step 4). | `references/docs-drift-check.md` |
| 7 | **G7** merge | Only when G1–G6 all ✓ and `mergeStateStatus == CLEAN`. | `references/merge.md` |

### The loop boundary — what you wait on and what you don't

This is the heart of the execution model, and it's a single principle: **loop on synchronous local work you control; never block on asynchronous remote state.**

- **Loop locally.** Running `lint --fix`, `tsc`, or a test suite *locally* gives an immediate result — fix, re-run, repeat until clean. That's work you control, so stay on it.
- **Push, then return — don't wait for remote CI.** Once you push, GitHub's CI is asynchronous and can take minutes. You cannot confirm it's green from inside this pass, so don't try. Report honestly: *"fixed X and pushed; CI is re-running, so I can't confirm everything's green until it completes."* Getting it this far is the win; verifying the remote outcome is the next invocation's job.
- **Return immediately on human waits.** A pending approval is indeterminate. Request the reviewer (the action), then stop (the wait).

You do **not** own scheduling. Don't sleep, busy-poll, or assume a particular harness feature exists. Do what's actionable, push, return your status. *Whether* to re-check later — via `/loop`, a scheduled sub-agent if the harness supports one, or a manual re-run — is the orchestrator's call. You may note that a re-check is warranted; you don't implement the wait.

---

## Step 4 — Documentation drift-check (non-blocking, run last)

Run this **after** the other work, because the fixes you just made (resolved comments, CI changes, a rebase) can leave the title/description describing a PR that no longer exists. Read `references/docs-drift-check.md`. It confirms three things:

1. Does the title/description still match the actual change?
2. Does the description capture **all** the behavioural changes?
3. Does the **title** carry a correct issue reference?

Make a **best-effort** fix where any of these fall short — but this **never blocks the merge**. An honest description is worth a quick pass; blocking an approved, green PR because an agent-drafted branch lacks a tracker ticket is exactly the kind of technicality that tempts an `--admin` override, which this skill refuses to do.

---

## Step 5 — Communicate

Default to **unattended** — no per-action confirmation prompts; automate the whole path. The only real decision is *where* each blocker's escalation goes, and the rule is: **route it to whoever can actually resolve it.**

- **Orchestrator-resolvable** (which reviewer to pick, whether a CI failure is in scope, an ambiguous conflict) → **terminal only**. The person who can unblock is the human running the skill; a PR comment would just be noise.
- **Collaborator-resolvable** (a review comment needing a decision) → **on the PR, to that person**: reply in-thread *and* `@`-mention them so they're pulled in. No orchestrator confirmation needed.
- **Never nag.** Request a reviewer once; don't leave "any update?" comments. Before posting anything to the PR, check you haven't already said the same thing — re-runs must not spam the thread.

Always emit this compact block in the terminal. No preamble. `[x]` pass, `[·]` fail:

```
## Done-Done: <owner>/<repo>#<N> — <title>

- [<x|·>] G1 Intent clear
- [<x|·>] G2 Mergeable
- [<x|·>] G3 Comments resolved
- [<x|·>] G4 CI green
- [<x|·>] G5 Ready for review
- [<x|·>] G6 Approved
- [<x|·>] G7 Merged
      Docs: <ok | drift fixed | n/a>

Did this pass: <one line per action taken, or "—">
Blockers (need a human): <who needs to do what, or "none">
Verdict: <MERGED | BLOCKED — needs human: <summary> | IN PROGRESS — CI re-running>
```

Append `— <reason>` (≤ 12 words) to any failing gate line, e.g. `G4 CI green — 2/7 failed: lint, e2e`.

The **Blockers** line is the end-of-pass summary: aggregate everything a human must resolve into one place so the operator sees their whole to-do at a glance.

When `merged == true`, emit only the checklist + `Verdict: MERGED at <mergedAt>` — nothing else, no actions this pass.

---

## Step 6 — Looping is the orchestrator's job, not yours

This skill does **one bounded work-pass** per invocation: clear what you can, push, report (it self-loops only on synchronous local work — Step 3). Re-driving across CI runs and review waits is done by wrapping it:

```
/loop 10m /done-done https://github.com/acme/repo/pull/123
```

`/loop` re-invokes each tick and terminates on the literal `Verdict: MERGED` token. It's **optional** — the skill is useful as a single pass — but it's the natural way to carry a PR across the waits the skill won't block on. If you change the verdict line, update both skills together.

A new gate must be expressible as a boolean over the `gh pr view` payload; anything needing scoring or weighting belongs in the action layer or in `pr-triage`.
