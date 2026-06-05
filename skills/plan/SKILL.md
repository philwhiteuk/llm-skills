---
name: plan
description: >
  Turn an approved spec into a terse, well-structured implementation plan that says exactly how the
  work will be built. Use this skill whenever the user wants an implementation plan, a technical
  approach, or a breakdown of how to build something they've already specced. Also trigger when the
  user says things like "plan out how to build this", "how should we implement X?", "write the
  implementation plan", "what's the technical approach?", "break down the work on PROJ-123", "turn
  this spec into a plan", or "how do we actually do this?". This is the step *after* the spec — where
  the spec captures what and why solution-agnostically, the plan owns the how. Use it even from rough
  notes or a bare ticket; you can read the codebase and shape a plan, asking the user any blocking
  questions before you write.
---

# Plan

You turn an approved spec into an implementation plan: a precise, scannable description of *how* the
work will be built. The spec is the mirror you work against — it deliberately excludes mechanism so
the team can own technical decisions. The plan is where that mechanism finally lands.

A good plan reads like a confident set of instructions, not a train of thought. An engineer (or the
user, scanning at handover) should know exactly where to look for each kind of detail and trust that
what's there is decided, not deliberated.

## The two disciplines

The plan earns its keep the same way the spec does — by being **constrained** and **decisive**.

**Carry as much detail as necessary, and no more.** The plan exists so nobody has to guess how to
build the thing. That means real file paths, real symbol names, concrete signatures and snippets
where they remove ambiguity. It does *not* mean narrating your reasoning. If a sentence explains
*why a decision is good* rather than *what to do*, it's cruft — cut it.

**Decide before you write; never leave questions in the output.** A plan with open questions isn't
finished — it's a plan-shaped list of things still to figure out. When the spec leaves something
genuinely ambiguous, resolve it *first* (see [Resolve ambiguity first](#resolve-ambiguity-first)),
then write a plan that commits to an answer.

## What the plan excludes

These belong to other documents or to nobody. Their absence is what keeps the plan scannable:

- **Status quo / background** — the reader has the spec; don't re-describe the current system.
- **Why / justification** — state the decision, not the case for it. "Rotate the token on refresh,"
  not "Rotating on refresh is better because…".
- **Tradeoffs and rejected alternatives** — the deliberation happened; the plan is the conclusion.
- **Risks considered** — a plan is a commitment to a path, not a survey of paths.
- **Open questions** — resolved before writing, never parked in the output.
- **Preamble** — no "Here's my thinking" / "Let me walk through the approach". Open with the plan.

If you catch yourself writing any of these, you've slipped from *planning* into *thinking out loud*.
The thinking is real and useful — just keep it in chat, not in the artefact.

## Resolve ambiguity first

Before writing, read the spec and the actual code, then ask: **would two reasonable engineers, given
this spec, build materially different things?** Each fork like that is a blocking question.

- **If yes — ask the user, then wait.** Batch the questions, keep them concrete, and offer the
  options you see. "Should existing sessions survive the migration, or is a forced re-login
  acceptable?" beats "How should I handle sessions?". Resolve every blocking fork before you write a
  line of plan.
- **If no — just decide.** Where a sensible, conventional default is obvious and low-stakes, take it
  silently. Don't ask permission for the unremarkable, and don't pad the plan with an "assumptions"
  list — that's just open questions wearing a hat. The plan states what will be built; if a quiet
  default was wrong, the user corrects it when they read the decisive plan.

The bar for asking is *impact*, not *certainty*. Ask when the answer changes the shape of the plan or
is expensive to reverse. Stay quiet when it doesn't.

## Ground the plan in the codebase

A plan made of guesses is worse than no plan. Before writing, read the relevant code so every
reference is real: existing modules the change extends, the patterns already in use, the names you'll
touch. The Breakdown should cite symbols and paths that actually exist. Match the conventions you
find rather than importing your own.

## Writing the plan

Read the template at `assets/plan.md` and fill it in — don't reconstruct it from memory; reproduce
the section headers and emoji exactly so the format stays recognisable across every plan. The
sections, and how to keep each one honest:

- **🎯 Approach** — one sentence naming the strategy. If it needs two, the plan may be doing two
  things; consider whether the spec should have been split.
- **🔧 Changes** — a table of the logical units of work, each with a one-line summary. This is the
  at-a-glance map; the reader should grasp the whole shape from this table alone.
- **🛠️ Breakdown** — one subsection per change from the table, in the same order. This is where
  detail lives: signatures, snippets, data shapes, call sequences — whatever an engineer needs to
  build it without guessing. Terse prose and code, not essays. Skip a change's subsection only if its
  one-liner in the table is genuinely complete.
- **📂 Files** — the physical surface area: every file created, modified, or deleted, one row each.
  The Changes table is logical (what we're doing); this is physical (where it lands). Both earn their
  place because they answer different questions a reader has.
- **🧪 Test Strategy** — how the work is proven, mapped to the spec's acceptance criteria. Name the
  level (unit / integration / e2e / manual) and the case, not a full test plan.

Keep the whole thing as short as the work allows. A trivial change might be three rows and a
sentence; a large one might have a dozen breakdown subsections. Let the work set the length — never
inflate to look thorough, never compress past the point of being buildable.

## After writing

If you're working an issue, the plan is destined for the tracker as a comment (the workflow skill
handles posting it). Otherwise, present it and ask: "Does this approach work, or is anything off?"
Hold revisions to the same disciplines — terse, decided, no open questions.
