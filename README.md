# skillz

A collection of general-purpose LLM skills that I use in my day-to-day development workflow. Each skill is a structured prompt that helps automate common engineering tasks like writing specifications and classifying work items. The prompts can be adapted for use with any LLM-powered coding tool.

## Skills

<!-- SKILLS-INDEX:START -->
| Skill | Description |
|-------|-------------|
| [spec](skills/spec/SKILL.md) | Write solution-agnostic specifications from raw input and classify them as a Story, Task, Refactor, Spike, Bug, or Epic — rendered in the correct standardised format for sharing with the team. Behaviour-preserving restructuring of existing code is a Refactor; net-new internal/infrastructure work is a Task. |
| [plan](skills/plan/SKILL.md) | Turn an approved spec into a terse, structured implementation plan that owns the *how* — Approach, a Changes table, a detailed Breakdown grounded in the real codebase, Files touched, and Test Strategy. Resolves shape-changing ambiguity by asking up-front, so the plan is decided rather than a wall of reasoning with open questions left in. The mechanical mirror of the spec. |
| [slack-message](skills/slack-message/SKILL.md) | Craft and send beautifully formatted Slack messages using the Slack MCP. Covers technical Q&A replies and PR notifications, with markdown structure, emojis as visual signposts, and guidance on including ticket links and finding recipients. |
| [dlc](skills/dlc/SKILL.md) | Orchestrate the full software delivery lifecycle — from logging an issue through refinement, implementation, and review. Determines the current step, drives work forward autonomously, and pauses only at human approval gates. |
| [pr-triage](skills/pr-triage/SKILL.md) | Triage a GitHub pull request to determine whether it's ready for human review. Evaluates hard gate checks (draft status, CI, spec clarity) and weighted heuristics (prior reviews, agentic code signals, blast radius). Ready PRs trigger a Slack DM; unready ones get a friendly human-sounding comment on the PR. |
| [done-done](skills/done-done/SKILL.md) | Drive a PR all the way to merged. Reads PR state in a single GitHub fetch, evaluates seven deterministic merge gates (intent clear, mergeable, comments resolved, CI green, ready, approved, merged), then works each failing gate with judgment — rebasing, fixing CI, resolving comments, requesting the right reviewer from file history — escalating only what needs a human. Documentation drift is fixed best-effort but never blocks. Never uses admin override or auto-merge — merges only when every gate genuinely passes. Pair with `/loop` to keep driving across CI runs and review waits. |
| [no-word-salad](skills/no-word-salad/SKILL.md) | A one-sentence self-discipline lens: write the minimum the task needs in plain words — no AI stock-phrases, hedging, or filler — and comment code only where intent isn't obvious, never to narrate it. Referenced by `dlc` at self-review as a second lens over the diff; also invokable directly to scrub a draft. |
<!-- SKILLS-INDEX:END -->

## Installation

Each skill is a self-contained markdown prompt in `skills/`. You can use them directly with any LLM by copying the prompt content, or install them as a plugin for supported tools.

### Claude Code

```sh
claude plugin add philwhiteuk/llm-skills
```

## License

[MIT](LICENSE)
