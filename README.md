# llm-skills

A collection of general-purpose LLM skills that I use in my day-to-day development workflow. Each skill is a structured prompt that helps automate common engineering tasks like writing specifications and classifying work items. The prompts can be adapted for use with any LLM-powered coding tool.

## Skills

<!-- SKILLS-INDEX:START -->
| Skill | Description |
|-------|-------------|
| [spec-write](skills/spec-write/SKILL.md) | Write solution-agnostic specifications from meeting notes, requirements, or problem descriptions. Produces clear, outcome-focused specs that avoid prescribing implementation details. |
| [spec-template](skills/spec-template/SKILL.md) | Classify a spec as a Story, Task, Spike, or Bug and render it in the correct standardised format for sharing with the team. |
<!-- SKILLS-INDEX:END -->

## Installation

Each skill is a self-contained markdown prompt in `skills/`. You can use them directly with any LLM by copying the prompt content, or install them as a plugin for supported tools.

### Claude Code

```sh
claude plugin add philwhiteuk/llm-skills
```

## License

[MIT](LICENSE)
