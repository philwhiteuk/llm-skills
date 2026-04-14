---
name: spec
description: >
  Write and format software specifications for an engineering team. Use this skill whenever the
  user wants to write a spec, draft requirements, turn meeting notes into a specification, create
  acceptance criteria, classify a piece of work, or format a spec as a ticket (Story, Task, Spike,
  Bug, or Epic). Also trigger when the user says things like "write me a spec", "make this a ticket",
  "what type of work is this?", "turn this into a story/task/spike/bug/epic", "format this for
  the team", "user stories", "acceptance criteria", or "requirements doc". Use this skill even
  if the user only has rough notes — you can help shape them into a spec.
---

# Spec

You help an Engineering Lead go from raw input to a formatted, shareable ticket in two steps:

1. **Write** — turn whatever the user gives you (notes, requirements, a problem description) into a solution-agnostic spec
2. **Classify & format** — classify the spec as a Story, Task, Spike, Bug, or Epic and render it using the right template from `assets/`

If the user already has a well-formed spec, skip straight to step 2. If they only have rough input, do step 1 first, then proceed.

---

## Step 1: Writing the spec

### Why solution-agnostic matters

A spec that names technologies constrains the team before they've had a chance to think. When the spec says "system responds within 200ms for repeated requests" instead of "implement Redis cache", the team can evaluate trade-offs and own their decisions.

### Principles

**Describe outcomes, not mechanisms.** Every requirement should be verifiable by observing system behaviour, not by inspecting internals. If you find yourself naming a technology, framework, or architecture pattern — stop and reframe.

| Implementation-specific | Solution-agnostic |
|---|---|
| Implement Redis cache | System responds within 200ms for repeated requests |
| Create REST API | External systems can integrate with our data programmatically |
| Build a React dashboard | Users can view key metrics at a glance from a browser |
| Add PostgreSQL full-text search | Users can search across all content by keyword with results in under 1 second |
| Use WebSockets for live updates | Changes made by one user are visible to others within 3 seconds without refreshing |

**Be vendor and tool agnostic** unless the requirement genuinely depends on a specific integration (e.g. "must authenticate against the company's existing Okta instance" — that's a constraint, not a choice).

**One spec, one problem.** If scope starts sprawling, suggest splitting rather than cramming everything in.

**Be concise.** A spec is a communication tool, not a novel. The Why should be 2-3 sentences. Out of Scope and Open Questions should be short bullet lists.

### Spec structure

```
### Title
Short outcome-focused name. Not a solution name.
Good: "Sub-second search across content library"
Bad: "Elasticsearch implementation"

### Why
2-3 sentences. What problem does this solve and why now?

### Who
One primary user story: As a [type of user], I want to [achieve something], so that [benefit].
Include 2-3 stories only if genuinely multiple distinct user types — and flag it may need splitting.

### What (Acceptance Criteria)
Numbered list of observable, testable outcomes. Each one should be demonstrable in a review.
- Start with a verb ("Users can...", "System displays...", "Data is available within...")
- Include measurable thresholds where relevant
- No technology names

### Out of Scope (optional)
Brief bullets on what this spec doesn't cover.

### Open Questions (optional)
3-4 max. Unresolved items that need investigation before the work is fully defined.
```

### How to work with the user

- **Actively reframe implementation language.** Don't just flag it — propose the solution-agnostic alternative and show your reasoning.
- **Ask clarifying questions** when the outcome isn't clear. "We need caching" → "What's slow today, and how fast does it need to be?"
- **Push back gently on scope creep.** Suggest splitting if the problem grows.
- After producing the spec, ask: "Does this capture the problem accurately? Anything missing or accidentally prescribing a solution?"

---

## Step 2: Classify and format

Once you have a spec (written above or supplied by the user), classify it and render it using the correct template.

### Classification rules

**Epic**
A large, strategic initiative spanning multiple features or teams. Too big to deliver in a single sprint — it will be broken into child Stories, Tasks, or Spikes. The goal is a significant product or business outcome, not a single user workflow.
> Signal: Multiple user stories are needed to describe the scope, or the work would clearly take many sprints. Language like "platform", "programme", "initiative", "phase", or explicit mention of sub-features.

**Story**
A new feature **added** to the product. The user story describes an end-user benefit, the ACs are user-facing and observable, and there are **minimal unknowns** — the team knows roughly what to build.
> Signal: "As a [end user], I want to..." with ACs describing visible behaviour. No significant Open Questions.

**Task**
Supporting or infrastructure work **in service of** a feature. The primary "user" is the system, a developer, or an internal team — not an end-user. No direct standalone user benefit; it enables something else.
> Signal: The "Who" is internal/technical, or the work is clearly a dependency for something bigger.

**Spike**
The **direction is uncertain**. Open questions, major assumptions, or competing approaches need investigation before real work can be scoped. Spikes are timeboxed research — the output is a finding or recommendation, not shipped software.
> Signal: Non-trivial Open Questions section, OR ACs that depend on knowing the approach first. Language like "evaluate", "determine", "assess", "investigate".

**Bug**
An **existing feature is broken** or behaving incorrectly. Corrective work, not additive.
> Signal: A discrepancy between intended and actual behaviour of something already shipped.

### Handling ambiguity

- **Story vs Task**: Does it deliver standalone value to an end-user? If no, it's a Task.
- **Story vs Spike**: Non-empty Open Questions, or ACs that depend on the implementation → lean Spike.
- **Epic vs Story**: Could this reasonably be done in one sprint by one team? If no, it's an Epic.
- **Spike vs Task**: A Spike answers a question; a Task completes a known piece of work.

If unsure, state your uncertainty and offer a recommendation. The user can override.

### Rendering

Read the appropriate template from `assets/` and fill in every `{{placeholder}}`. Reproduce all section headers and emoji exactly as written.

**Epic** (`assets/epic.md`):
- `{{SUMMARY}}` → spec Title
- `{{objective}}` → a concise statement of what this initiative sets out to achieve; this is the record of original intent — write it to be meaningful even if nothing else about the epic ever gets updated (1-2 sentences, outcome-focused)
- `{{why}}` → from the spec's Why section; the problem being solved and why it matters now (2-3 sentences, plain text)
- `desired_outcomes` → the observable, measurable results that would signal success; derive from the What section or synthesise from the spec's intent. These are not deliverables — they are the changes in the world the initiative should produce (e.g. "customers can self-serve without contacting support", "deployment frequency increases without a rise in incidents"). Bullet list, 2-4 items.

**Story** (`assets/story.md`):
- `{{SUMMARY}}` → spec Title
- `{{persona}}` / `{{goal}}` / `{{benefit}}` → parse from the Who section user story
- `acceptance_criteria` → from What section, as bullet points
- `discussion_points` → from Open Questions (omit the block entirely if absent)

**Task** (`assets/task.md`):
- `{{SUMMARY}}` → spec Title
- `{{task_description}}` → what needs to be done (synthesise from the spec; reframe as a task statement, not a wish)
- `acceptance_criteria` → from What section
- `{{context}}` → from Why section (1-2 sentences, plain text)

**Spike** (`assets/spike.md`):
- `{{SUMMARY}}` → spec Title, prefixed with "Spike: " if not already
- `{{spike_description}}` → what needs to be investigated and why; frame as a question to answer
- `{{context}}` → from Why section (plain text)
- `{{output_1}}`, `{{output_2}}` → concrete deliverables the spike should produce; derive from Open Questions and ACs

**Bug** (`assets/bug.md`):
- `{{SUMMARY}}` → spec Title
- `{{expected}}` → what the system *should* do
- `{{actual}}` → what the system *currently* does incorrectly
> If the spec doesn't cleanly separate expected vs actual, extract what you can and flag it for the user to verify.

### After rendering

Show the rendered output as a Markdown code block so it's easy to copy. Then say:

> "Classified as a **[Type]** because [brief reason — 1-2 sentences]. Does this look right, or would you like to reclassify or adjust anything?"

Don't ask more than one follow-up question. Keep it tight.
