---
name: verify-docs
description: Validate JSON, file paths, and settings keys across all docs
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash(node -e *)
---

# Verify Docs

Validate all documentation files in the repo for correctness.

## Steps

1. **Collect files:** Glob all `.md` files in these directories:
   - `core/`
   - `projects/`
   - `templates/`
   - `skill/`
   - `.claude/skills/`
   - `.claude/rules/`

2. **Validate JSON code blocks:** For each file, extract all ` ```json ` fenced code blocks and validate each one:
   - Use `Bash(node -e 'JSON.parse(require("fs").readFileSync("/dev/stdin","utf8"))' < tmpfile)` or inline the JSON string
   - Flag: trailing commas, comments (`//` or `/* */`), single quotes, unquoted keys
   - Record file path and line number of each invalid block

3. **Validate file paths:** Extract all relative file paths referenced in the docs (patterns like `core/`, `.claude/`, `projects/`, `templates/`, `skill/`):
   - Grep for path-like references
   - Verify each referenced file or directory actually exists using Glob
   - Flag broken references with file and line number

4. **Validate settings keys:** Extract settings keys used in JSON examples (e.g., `permissions`, `allow`, `deny`, `hooks`, `mcpServers`):
   - Cross-check against `skill/SKILL.md` Section 4 (Config Reference) as the source of truth
   - Flag any key not present in the reference

5. **Validate hook events:** Extract hook event names (e.g., `PreToolUse`, `PostToolUse`, `Stop`, `SubagentStop`):
   - Cross-check against known valid events in `core/hooks-guide.md`
   - Flag unrecognized events

6. **Report:** Group all issues by file. For each issue, show:
   - File path and line number
   - Issue type: `INVALID_JSON`, `BROKEN_PATH`, `UNKNOWN_KEY`, `UNKNOWN_EVENT`
   - Description of the problem
   - If no issues found, report: "All docs passed verification."
