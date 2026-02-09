# BOOTSTRAP.md — Auto-Configure Claude Code for Your Project

This file previously contained the full bootstrap sequence. It has been converted to a skill for on-demand loading.

## Usage

Run the bootstrap skill in Claude Code:

```
/bootstrap
```

This will interactively:
1. Detect your project stack (language, framework, tools)
2. Generate a tailored `CLAUDE.md`
3. Configure `.claude/settings.json` with permissions and hooks
4. Create skills (`/fix-issue`, `/code-review`, and conditional ones)
5. Create custom agents (code-reviewer, test-writer)
6. Set up MCP servers (GitHub, database, Slack, etc.)
7. Generate `.claude/rules/` for code style, testing, and git conventions

## Skill Location

The full bootstrap logic lives in `.claude/skills/bootstrap/SKILL.md`.

## When to Use

Use `/bootstrap` when setting up Claude Code on a new or existing project for the first time. It replaces manual configuration with an interactive, stack-aware setup flow.
