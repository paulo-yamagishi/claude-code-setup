# CLAUDE.md — Meta Instructions

This repository's primary artifact is `skill/SKILL.md` — a `/setup` skill that audits Claude Code configuration and suggests project-tailored improvements. The `core/` directory serves as reference material for maintaining the skill's accuracy. `BOOTSTRAP.md` is a secondary tool for greenfield project setup.

When working on this repo:

## Content Standards
- All guides must be accurate against official docs at code.claude.com/docs
- Keep each file focused on ONE topic
- Use concrete examples and code blocks, not vague descriptions
- Cross-reference other files with relative paths: `../core/hooks-guide.md`
- Templates must be copy-paste ready with clear placeholder markers

## Structure Rules
- `core/` files are language-agnostic — no project-specific content
- `projects/` files extend core guides with language-specific details
- `templates/` files use `<!-- CUSTOMIZE -->` markers for editable sections
- Never duplicate content — import or link instead

## Style
- Markdown with GitHub-flavored extensions
- Headers: `#` for title, `##` for sections, `###` for subsections
- Code blocks with language tags (```json, ```bash, etc.)
- Tables for structured comparisons
- Keep files under 300 lines where possible

## Skill Rules (skill/SKILL.md)
- Written as imperative instructions TO Claude, not documentation
- Must be self-contained — no @imports from this repo (users install the skill standalone)
- Config formats must match: `core/hooks-guide.md`, `core/settings-reference.md`, `core/mcp-guide.md`, `core/subagents-guide.md`, `core/skills-guide.md`
- All JSON must be valid — no comments
- Keep under 250 lines
- Audit checklist: 3 checks per category max

## Verification
- All JSON examples must be valid JSON
- All file paths must be correct relative paths
- Settings keys and hook events must match official documentation
