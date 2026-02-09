---
name: refactor-docs
description: Split oversized or multi-topic docs into focused files with progressive disclosure
argument-hint: "[file-or-directory]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
---

# Refactor Docs

Split oversized or multi-topic documentation files into focused, smaller files following progressive disclosure principles.

## Steps

1. **Determine scope:**
   - If `$ARGUMENTS` is a file path, analyze only that file
   - If `$ARGUMENTS` is a directory, analyze all `.md` files in it
   - If `$ARGUMENTS` is empty, scan all `.md` files in: `core/`, `projects/`, `templates/`, `skill/`, `.claude/skills/`, `.claude/rules/`

2. **Identify candidates:** For each file, flag if:
   - Over 300 lines
   - Covers multiple distinct topics (detected by counting `##` sections on unrelated subjects)
   - Has large reference sections that could be extracted

3. **Propose split strategy** for each flagged file:

   **Core guides:** Split into `{topic}-guide.md` (essentials, <150 lines) + `{topic}-reference.md` (deep-dive details, examples, edge cases)

   **CLAUDE.md files:** Move detailed sections to separate files importable via `@import`, keep CLAUDE.md as a concise overview (<100 lines)

   **Rule files:** Break multi-topic rule files into single-topic files, each with appropriate `paths` frontmatter for scoped loading

   **Skill files:** Extract large reference sections into companion files only if the skill exceeds 250 lines (skills should be self-contained where possible)

4. **Present the plan** with before/after line counts for each file. Wait for user approval before proceeding.

5. **Execute the refactor:**
   - Create new files with extracted content
   - Update the original file to reference the new files
   - Update `@import` directives in CLAUDE.md if applicable
   - Update cross-references in other files that linked to the original

6. **Verify:** After refactor, confirm:
   - No broken cross-references (Grep for old paths, check they've been updated)
   - All new files are under 300 lines
   - No content was lost in the split
