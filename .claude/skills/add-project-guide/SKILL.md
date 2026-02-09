---
name: add-project-guide
description: Create a new projects/ language-stack guide with templates
argument-hint: "[language-or-stack]"
allowed-tools:
  - Read
  - Write
  - Glob
  - Grep
---

# Add Project Guide

Create a new `projects/{stack}/` directory with guide and templates for the language/stack specified in `$ARGUMENTS`.

## Steps

1. **Validate:** Confirm `projects/$ARGUMENTS/` does not already exist. If it does, report and stop.

2. **Study existing project guide:** Read the contents of one existing project directory (prefer `projects/python/` or `projects/typescript/`) to extract the pattern:
   - `guide.md` — stack-specific tooling, conventions, common commands, tips
   - `CLAUDE.md.template` — project instructions template with `<!-- CUSTOMIZE -->` markers
   - `settings.json.template` — settings with stack-appropriate permissions and hooks

3. **Generate `projects/$ARGUMENTS/guide.md`:**
   - Follow the same structure as the reference guide
   - Include stack-specific: package manager, test runner, linter, formatter, build tool
   - Include common commands and conventions for the stack
   - Add cross-references to relevant core guides
   - Keep under 300 lines

4. **Generate `projects/$ARGUMENTS/CLAUDE.md.template`:**
   - Use `<!-- CUSTOMIZE -->` markers for project-specific values
   - Include stack-appropriate sections: commands, code style, testing conventions
   - Follow the structure from the reference template

5. **Generate `projects/$ARGUMENTS/settings.json.template`:**
   - Include stack-appropriate permission patterns (test, lint, format, build commands)
   - Include appropriate hooks (format-on-save using the stack's formatter)
   - All JSON must be valid — no comments, no trailing commas

6. **Verify:** Re-read all generated files and confirm valid JSON, correct structure, under 300 lines each.
