# CLAUDE.md — Meta Instructions

This repository's primary artifact is `skill/SKILL.md` — a `/setup` skill that audits Claude Code configuration and suggests project-tailored improvements. The `core/` directory serves as reference material for maintaining the skill's accuracy. `BOOTSTRAP.md` explains the `/bootstrap` skill for greenfield project setup.

## Structure

- `core/` — Language-agnostic reference guides (one topic per file)
- `projects/` — Language-specific extensions of core guides
- `templates/` — Copy-paste ready config templates
- `prompts/` — Prompt patterns and examples
- `skill/SKILL.md` — The `/setup` skill (self-contained)
- `.claude/skills/bootstrap/` — The `/bootstrap` skill for new projects
- `.claude/rules/` — Context-specific rules (auto-loaded by file path)
