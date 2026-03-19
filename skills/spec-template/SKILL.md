---
name: spec-template
description: >
  Classify a spec as a Story, Task, Spike, or Bug and render it in the correct standardised
  format for sharing with the team. Use this skill whenever the user has a spec (or spec-like
  content) and wants to classify the type of work or format it for their team.
  Also trigger when the user says things like "make this a ticket", "what type of work is this?",
  "turn this into a story/task/spike/bug", or "format this for the team".
---

# Spec Classifier

You are helping an Engineering Lead classify a spec and render it in the appropriate standardised format for sharing with their team.

## Your job

1. Read the spec
2. Decide which change type fits best (Story, Task, Spike, or Bug) — and explain your reasoning briefly
3. Render the output using the correct template from `assets/`
4. Invite the user to confirm or override the classification

Keep your explanation of the classification to 2-3 sentences. Don't over-justify — the user understands their domain well.

---

## Classification rules

### Story
A new feature being **added** to the product. The user story describes an end-user benefit, the acceptance criteria are user-facing and observable, and there are **minimal unknowns** — the team knows roughly what to build.

> Signal: "As a [end user], I want to..." with ACs that describe visible behaviour. No significant Open Questions section.

### Task
Supporting or infrastructure work **in service of** a feature. Often more technical — the primary "user" is the system, a developer, or an internal team rather than an end-user. There's no direct, standalone user benefit; it enables something else.

> Signal: The "Who" is internal/technical, or the work is clearly a dependency for something bigger (e.g., "create the database schema", "set up the data pipeline"). No major unknowns.

### Spike
The **direction is uncertain**. There are open questions, major assumptions, or competing approaches that need to be investigated before real work can be scoped. Spikes are timeboxed research tasks — the output is a finding or recommendation, not shipped software.

> Signal: The spec has a non-trivial Open Questions section, OR the "Why" describes uncertainty, OR the ACs would be hard to verify without first knowing the approach. Language like "evaluate", "determine", "assess", "investigate" is a strong indicator.

### Bug
An **existing feature is broken** or behaving incorrectly. This is corrective work, not additive. Something that once worked (or was intended to work) doesn't. There is no new user story — the user story is the one that already exists.

> Signal: The spec describes a discrepancy between intended and actual behaviour of something already shipped.

---

## Handling ambiguity

Because specs always include a user story and ACs, the signals aren't always obvious. When it's genuinely unclear:

- **Story vs Task**: Ask yourself whether the spec delivers standalone value to an end-user, or whether it's scaffolding for something else. If it's scaffolding, it's a Task.
- **Story vs Spike**: If the Open Questions section is non-empty, or if the ACs depend on knowing the implementation first, lean toward Spike. A Story should be ready to build.
- **Spike vs Task**: A Spike answers a question; a Task completes a known piece of work.

If you're still not sure, state your uncertainty and offer a recommendation with your reasoning. The user can override.

---

## Rendering

After classifying, read the appropriate template from `assets/` and use it as the exact structure for the output. Reproduce the template faithfully — including all section headers exactly as written, with their emoji characters. Fill in every `{{placeholder}}` with content drawn from the spec.

### Mapping spec → template fields

**Story** (`assets/story.md`):
- `{{SUMMARY}}` → the spec Title (used as the `# Heading`)
- `{{persona}}` / `{{goal}}` / `{{benefit}}` → parse from the spec's Who section user story
- `acceptance_criteria` → from the spec's What section, as bullet points
- `discussion_points` → from the spec's Open Questions section (omit the block entirely if absent)

**Task** (`assets/task.md`):
- `{{SUMMARY}}` → the spec Title
- `{{task_description}}` → a plain description of what needs to be done (synthesise from the spec; don't copy the user story verbatim — reframe as a task statement, not a wish)
- `acceptance_criteria` → from the spec's What section
- `{{context}}` → from the spec's Why section (1-2 sentences, plain text — no italics, no label prefix)

**Spike** (`assets/spike.md`):
- `{{SUMMARY}}` → the spec Title, prefixed with "Spike: " if not already
- `{{spike_description}}` → what needs to be investigated and why; frame as a question to answer, not a feature to build (draw from Why + Who + What)
- `{{context}}` → from the spec's Why section (plain text — no italics, no label prefix)
- `{{output_1}}`, `{{output_2}}` → the concrete deliverables the spike should produce; derive from Open Questions and ACs where possible. These populate the Acceptance Criteria section — they are what "done" looks like for this specific spike.

**Bug** (`assets/bug.md`):
- `{{SUMMARY}}` → the spec Title
- `{{expected}}` → what the system *should* do (from the spec's intended behaviour)
- `{{actual}}` → what the system *currently* does incorrectly (from the spec's problem description)

> If this is a Bug and the spec doesn't cleanly separate expected vs actual, extract what you can and flag it for the user to verify.

---

## After rendering

Show the rendered output as a Markdown code block so it's easy to copy. Then say something like:

> "Classified as a **[Type]** because [brief reason]. Does this look right, or would you like to reclassify or adjust anything?"

Don't ask more than one follow-up question. Keep it tight.
