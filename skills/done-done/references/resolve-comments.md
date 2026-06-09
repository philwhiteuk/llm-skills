# G3 — resolve outstanding comments

Gate G3 fails while any review thread is unresolved. This is a collaborative process: **treat every commenter equally** — the drafting agent's own self-review notes, a teammate's feedback, a senior's nitpick. An unresolved actionable comment is an unresolved actionable comment, whoever left it. Give all of them priority.

## 1. Fetch unresolved threads

The `gh pr view` payload doesn't carry resolved-state; get it from GraphQL:

```bash
gh api graphql -f query='
  query($owner:String!,$repo:String!,$num:Int!){
    repository(owner:$owner,name:$repo){
      pullRequest(number:$num){
        reviewThreads(first:100){ nodes{
          id isResolved isOutdated
          comments(first:10){ nodes{ databaseId author{login} body path line } }
        } }
      }
    }
  }' -F owner=<owner> -F repo=<repo> -F num=<N>
```

Work the threads where `isResolved == false`.

## 2. For each unresolved thread, action or escalate

**Clear, mechanical fix** (typo, naming, null check, missing test case, style, a TODO the commenter flagged) — apply it, then commit and push with the rest of the comment fixes:

```bash
git add -A && git commit -m "fix: address review feedback on <thing>" && git push
```

Reply to the thread saying what you did, then **mark it resolved** — because you actually addressed it, closing it is honest:

```bash
# reply
gh api -X POST repos/<owner>/<repo>/pulls/<N>/comments/<comment-id>/replies -f body="Done — <what you changed>."
# resolve (GraphQL mutation; needs the thread node id, not the comment id)
gh api graphql -f query='mutation($id:ID!){ resolveReviewThread(input:{threadId:$id}){ thread{ isResolved } } }' -F id=<thread-node-id>
```

Only resolve a thread you genuinely addressed. Never resolve one you haven't, just to clear the gate — that reads as dismissive and hides real feedback.

**Needs a decision** (design question, scope call, anything ambiguous) — do **not** guess; guessing on someone else's PR causes thrash. This is collaborator-resolvable, and the owner is the commenter, not the operator. Reply in-thread **and** `@`-mention them so they're notified, then **leave the thread open**:

```bash
gh api -X POST repos/<owner>/<repo>/pulls/<N>/comments/<comment-id>/replies \
  -f body="@<commenter> this needs your call: <the specific question>. Happy to apply either way once you confirm."
```

## 3. Re-request review after change-requests

If a reviewer left "changes requested" and you've now addressed their points, that block won't clear until they look again — so re-request them. It's a legitimate mechanical signal (they asked, you delivered), and it fits automating the path:

```bash
gh pr edit <N> --add-reviewer <reviewer-login>
```

Don't `@`-mention to nag beyond this single re-request.

## Report back

A short summary with counts: how many unresolved threads, how many you fixed-and-resolved, how many you left open awaiting a commenter's decision (and who), whether you re-requested any reviewer, and whether G3 now passes.
