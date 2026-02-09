---
name: update-readme
description: Sync README.md with current skill capabilities and repo state
allowed-tools:
  - Read
  - Edit
  - Glob
  - Grep
---

# Update README

Sync the README.md with the current state of `/setup` and `/bootstrap` skills.

## Steps

1. **Read current skill definitions:**
   - Read `skill/SKILL.md` — extract all audit categories, checks per category, and feature descriptions
   - Read `.claude/skills/bootstrap/SKILL.md` — extract all phases, capabilities, and supported stacks

2. **Read README.md** and identify sections that reference skill features:
   - "What Gets Audited" table or list
   - Usage examples and command syntax
   - Feature descriptions and capability claims
   - Phase/step counts for bootstrap

3. **Diff skill state vs README:**
   - Flag any audit category in the skill but missing from README
   - Flag any check count that doesn't match
   - Flag any bootstrap phase or capability not reflected in README
   - Flag any removed feature still mentioned in README

4. **Update README sections:**
   - Edit only the stale sections — do not rewrite unrelated content
   - Preserve existing markdown style: tables, code blocks, heading levels
   - Keep descriptions concise — match the existing tone

5. **Report changes** — list each section updated with a one-line summary of what changed.
