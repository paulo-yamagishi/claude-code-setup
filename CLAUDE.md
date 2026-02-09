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

## Architecture

This is a **documentation and skills repository** — no application code, no build step, no tests. The primary artifacts are Markdown files that serve as Claude Code configuration guides and skills.

### Key Files
- `skill/SKILL.md` — The `/setup` skill (also mirrored at `.claude/skills/setup/SKILL.md`)
- `.claude/skills/bootstrap/SKILL.md` — The `/bootstrap` skill
- `skill/OUTPUT-FORMAT.md` — Output template for `/setup` reports

### Data Flow
- `/setup` audits → emits `setup-report` block → `/bootstrap` consumes it
- `core/` guides are reference material for skill accuracy, not user-facing docs

## Commands

No build, test, or lint commands — this is a pure Markdown repo. Useful commands:

```bash
# Validate docs
/verify-docs

# Audit this project's Claude Code config
/setup

# Apply suggested config
/bootstrap
```

## Code Style

- **Markdown:** GitHub-flavored, `#` for title, `##` for sections, `###` for subsections
- **JSON examples:** Must be valid JSON — no comments, no trailing commas
- **File paths:** Always correct relative paths from repo root
- **Line limits:** Files under 300 lines where possible; skills under 200 lines
- **Cross-references:** Use relative paths like `../core/hooks-guide.md`

## Skill Lifecycle

After completing a major task, evaluate:
- **Create a new skill** if the task involved a repeatable workflow that would benefit from a dedicated `/skill-name` command
- **Update an existing skill** if you used a skill and found it missing steps, producing suboptimal results, or not covering edge cases you encountered
