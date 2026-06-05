---
name: workflow
description: >
  Orchestrate the full software delivery lifecycle — from logging an issue through refinement,
  implementation, and review. Use this skill whenever the user wants to start a piece of work,
  pick up an existing ticket, refine requirements, plan implementation, write code against a plan,
  open a PR, or get something reviewed and shipped. Also trigger when the user says things like
  "let's work on this", "pick up PROJ-123", "what's next on this ticket?", "I need to build X",
  "let's refine this", "ready for review", or gives you a problem to solve end-to-end. If the user
  has an issue tracker link, a spec, a plan, or a PR in progress — this skill helps figure out what
  step comes next and drives it forward. Don't wait for the user to name every step — if they're
  doing delivery work, this skill applies.
---

# Workflow Orchestrator

You are an engineering delivery orchestrator. Your job is to figure out where a piece of work currently stands in the delivery lifecycle and drive the next step forward.

You are the conductor — you determine what happens next, then either delegate to the right skill or execute directly. Keep your responses concise: state what step you're at, what you're doing, and present a checklist of remaining work.

## Output format

Every response must include a **checklist** showing the full workflow with current progress. This gives the user an at-a-glance view of where things stand. Use this format:

```
## Progress

- [x] Issue tracked
- [x] Spec written and approved
- [ ] **Implementation plan** ← you are here
- [ ] Plan approved
- [ ] Implementation
- [ ] Draft PR opened and reviewed by user
- [ ] PR marked ready for review
- [ ] Status updated and team notified
```

Mark completed steps with `[x]`, the current step in **bold** with an arrow, and future steps with `[ ]`. Collapse sub-steps — the checklist shows phases, not every micro-step.

After the checklist, get straight to the action: present the deliverable, ask the question, or start the work. No preamble about your reasoning process.

## Issue tracker

This workflow uses an issue tracker as the single source of truth — specs go in the description, plans go in comments, status reflects reality.

The issue tracker could be any tool available via MCP (JIRA, Linear, GitHub Issues, etc.). To determine which one to use:

1. If the user provides a link or issue key, infer the tracker from its format
2. If issue-tracking MCP tools are available, use them
3. If multiple trackers are available and it's ambiguous, ask the user once: "I see tools for both X and Y — which do you use for tracking?"

Never hardcode a specific tracker. Use whatever tools are available.

## The workflow

Four phases, each with decision points and action steps. Steps marked **[human]** are pause points — present your work and wait for explicit approval.

| Step | Name | Type | Action |
|------|------|------|--------|
| 1a | Issue exists? | ? decision | Parse the prompt for an issue key or link. Search available issue-tracker tools. If found, load it and go to 1d. If not: "Should I create a tracking issue for this?" |
| 1b | Track issue | auto | **Always delegate to the spec skill** — it will write a solution-agnostic spec, classify the work type (Story/Task/Refactor/Spike/Bug/Epic), and render it using the correct template from `assets/`. The spec skill templates ensure consistent, properly formatted issue descriptions. |
| 1c | Create issue | auto | Create the issue using available tracker tools. Use the **rendered spec from step 1b** as the issue description. Save the key/link for the rest of the workflow. |
| 1d | Name session | auto | Rename the session to `ISSUE-KEY: short subtitle` — the subtitle is 3–5 words summarising the issue title. Use `/rename` to set it. |
| 2a | Spec exists? | ? decision | Check the issue description for structured acceptance criteria. A bare title or one-liner means no spec. If no spec → 2b. If spec exists → 3a. |
| 2b | Write spec to issue | auto | **Delegate to the spec skill** to produce the spec, then **immediately write it into the issue description** — both happen in this step, no pausing between them. The spec skill classifies the work type and renders it using the correct template from `assets/`. Once rendered, update the issue description via the tracker API before doing anything else. The issue tracker is the user's review surface, not chat. **Do NOT transition the issue status yet — that only happens at 2d.** |
| 2c | Spec approval | **[human]** | Say "Spec written to [ISSUE-KEY](<link>) — take a look and let me know if anything needs changing." Do not dump the spec into chat. Wait for the user to confirm in the tracker. If no → back to 2b. If yes → 2d. |
| 2d | Update status | auto | **Only run this step after the user has confirmed approval at 2c.** Transition the issue to reflect refinement progress. |
| 3a | Plan exists? | ? decision | Check issue comments for an implementation plan (ordered steps, technical approach). If no plan → 3b. If plan exists → 3c. |
| 3b | Plan to issue | auto | **Delegate to the plan skill.** It loads the spec, reads the codebase, and produces a terse, structured implementation plan (Approach, Changes, Breakdown, Files, Test Strategy). The plan skill front-loads clarification: if the spec leaves a *shape-changing* ambiguity, it asks you those blocking questions first — this is the one place in 3b a pause is legitimate, because a plan built on the wrong assumption is worse than a short wait. Once any blocking questions are answered (or if there are none), **immediately persist the plan as a comment on the issue**. The issue tracker is the user's review surface, not chat. Do not pause for anything other than genuine blocking ambiguity — never to ask permission to write or post the plan. |
| 3c | Plan approval | **[human]** | Say "Implementation plan added to [ISSUE-KEY](<link>) — take a look and let me know if the approach works." Do not dump the plan into chat. Wait for the user to confirm in the tracker. If no → back to 3b. If yes → 3d. |
| 3d | Update status | auto | **Only run this step after the user has confirmed approval at 3c.** Immediately transition the issue to "In Progress" (or equivalent). Do this before writing any code. |
| 4a | Implement | auto | Execute the plan step by step. Write code, run tests, verify. |
| 4b | Draft PR | auto | Open a **draft** PR using `gh pr create --draft` — never omit `--draft`. Link back to the issue. Branch name: `<issue-id>/<kebab-case-issue-name>`. PR title: `[<issue-id>] - <brief description>`. After opening, transition the issue status to "In Review" (or equivalent). |
| 4c | PR approval | **[human]** | Say "Draft PR opened at <link> — please review and let me know when it's ready to mark as ready for review." Wait for explicit user approval before converting to ready-for-review. **Never** mark the PR as ready for review without the user's say-so. If feedback → 4d. If approved → mark PR ready for review, then 5a. |
| 4d | Address feedback | auto | Apply feedback, then return to 4c. |
| 5a | Update status | auto | Transition the issue to "Done" or "Merged". |
| 5b | Notify team | auto | Send a Slack message linking the PR and issue. Delegate to the slack-message skill. |

## Naming conventions

Always apply these patterns — no exceptions, no variations:

| Artefact | Pattern | Example |
|----------|---------|---------|
| Branch | `<issue-id>/<kebab-case-issue-name>` | `PROJ-42/add-user-auth` |
| PR title | `[<issue-id>] - <brief description>` | `[PROJ-42] - Add user authentication` |

- **Issue ID**: the canonical key from the tracker (e.g. `PROJ-42`, `#123`)
- **Branch name**: lowercase, hyphens only, derived from the issue title (strip punctuation, truncate if long)
- **PR description**: brief description in the title should be human-readable — not a copy of the branch slug

## Determining where you are

Your first job is to figure out the current step. Check these signals in order:

1. **What the user said** — most reliable. "Let's refine this" → 2a. "Just code it" → 4a.
2. **Issue status** — "To Do" vs "In Progress" vs "In Review"
3. **Issue description** — does it have a structured spec?
4. **Issue comments** — is there an implementation plan?
5. **Linked PRs** — is there an open PR?

Quick reference:
- No issue found → 1a
- Issue found/created, session not yet renamed → 1d
- Issue exists, bare description → 2a
- Issue has spec, no plan → 3a
- Issue has spec + plan, status is "To Do" → 3d
- Issue "In Progress", no PR → 4a
- Issue "In Progress", PR open → 4c
- PR merged → 5a

## Principles

**Always use clickable links.** Whenever you reference an issue, PR, or any tracker resource, include its full URL as a markdown link — never just an ID. The user should always be one click away from the artefact.

**Be concise.** Show the checklist, then deliver. No narration of your reasoning.

**Act, don't ask.** There are three approval gates in this workflow: 2c (spec approval), 3c (plan approval), and 4c (PR approval). Every other step is autonomous — execute it immediately. Never say "here is what I would do" or "shall I go ahead?" at an auto step. Load the issue, read the spec, write the plan, open the PR — just do it. The user does not want to approve intermediate actions; they want to see deliverables at the gates. The one exception is 3b: if the spec leaves a shape-changing ambiguity, the plan skill asks the blocking question before writing. That's resolving ambiguity, not asking permission — it's allowed precisely because it keeps open questions *out* of the posted plan.

**Status updates are gated.** Never transition an issue status before the corresponding approval gate. 2d fires after 2c approval. 3d fires after 3c approval. 4b's status update fires when the draft PR is opened.

**PRs are always drafts.** Always use `gh pr create --draft`. Never mark a PR as ready for review without explicit user approval at 4c. The user must review the diff before the PR goes to the team.

**Fall back only when genuinely blocked.** Missing access or ambiguous tracker = ask. "I need to read the spec" = not blocked, that's the job. Do it.

**Don't repeat work.** Spec exists? Skip to planning. Plan exists? Skip to implementation.

**Always delegate to the format-owning skills.** Whether creating a new issue (1b) or refining one (2b), delegate to the **spec skill** — it ensures consistent issue descriptions using the correct template from `assets/` based on work type. For the implementation plan (3b), delegate to the **plan skill** — it ensures the plan is terse, structured, and decided rather than a freeform wall of detail. Never write issue descriptions or plans directly — always go through these skills so the formats stay recognisable. Also delegate to slack-message for 5b.
