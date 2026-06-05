# skillz

A collection of general-purpose LLM skills that I use in my day-to-day development workflow. Each skill is a structured prompt that helps automate common engineering tasks like writing specifications and classifying work items. The prompts can be adapted for use with any LLM-powered coding tool.

## Skills

<!-- SKILLS-INDEX:START -->
| Skill | Description |
|-------|-------------|
| [spec](skills/spec/SKILL.md) | Write solution-agnostic specifications from raw input and classify them as a Story, Task, Refactor, Spike, Bug, or Epic — rendered in the correct standardised format for sharing with the team. Behaviour-preserving restructuring of existing code is a Refactor; net-new internal/infrastructure work is a Task. |
| [slack-message](skills/slack-message/SKILL.md) | Craft and send beautifully formatted Slack messages using the Slack MCP. Covers technical Q&A replies and PR notifications, with markdown structure, emojis as visual signposts, and guidance on including ticket links and finding recipients. |
| [workflow](skills/workflow/SKILL.md) | Orchestrate the full software delivery lifecycle — from logging an issue through refinement, implementation, and review. Determines the current step, drives work forward autonomously, and pauses only at human approval gates. |
| [my-issues](skills/my-issues/SKILL.md) | Fetch and display a quick summary of all your active issues across every board and project. Shows a flat table with project, parent, issue ID, title, status, reporter, and labels — sorted by status urgency so the most actionable work appears first. |
| [pr-triage](skills/pr-triage/SKILL.md) | Triage a GitHub pull request to determine whether it's ready for human review. Evaluates hard gate checks (draft status, CI, spec clarity) and weighted heuristics (prior reviews, agentic code signals, blast radius). Ready PRs trigger a Slack DM; unready ones get a friendly human-sounding comment on the PR. |
| [done-done](skills/done-done/SKILL.md) | Drive a PR all the way to merged. Evaluates six deterministic gates (code complete, checks green, ready for review, peer approved, documented, merged) in a single GitHub fetch, then orchestrates one focused sub-agent per failing gate to clear it — CI fixes, doc writing, reviewer requests, and merge run in parallel where dependencies allow. Resolves author self-review comments before marking ready for peer review. Selects reviewers intelligently from file history rather than picking arbitrarily. Never uses admin override or auto-merge — merges only when all gates genuinely pass. Pair with `/loop` to keep driving across CI runs and review waits. |
<!-- SKILLS-INDEX:END -->

## Installation

Each skill is a self-contained markdown prompt in `skills/`. You can use them directly with any LLM by copying the prompt content, or install them as a plugin for supported tools.

### Claude Code

```sh
claude plugin add philwhiteuk/llm-skills
```

## License

[MIT](LICENSE)
