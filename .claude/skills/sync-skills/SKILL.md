---
name: sync-skills
description: Check /setup and /bootstrap skills for consistency with core guides
allowed-tools:
  - Read
  - Glob
  - Grep
---

# Sync Skills

Cross-reference the two main skills against core guides to find inconsistencies.

## Steps

1. **Read source of truth:**
   - `skill/SKILL.md` Section 4 (Config Reference) — canonical config formats
   - `core/hooks-guide.md` — hook events, matcher syntax, environment variables
   - `core/settings-reference.md` — settings structure, permission patterns
   - `core/skills-guide.md` — skill frontmatter format, allowed-tools syntax
   - `core/mcp-guide.md` — MCP server config format

2. **Read skills to check:**
   - `skill/SKILL.md` — the `/setup` skill (full file)
   - `.claude/skills/bootstrap/SKILL.md` — the `/bootstrap` skill (full file)

3. **Cross-reference config formats:**
   - Compare every JSON template in bootstrap against the canonical format in setup Section 4
   - Check hook event names match `core/hooks-guide.md`
   - Check settings keys match `core/settings-reference.md`
   - Check skill frontmatter fields match `core/skills-guide.md`
   - Check MCP config structure matches `core/mcp-guide.md`

4. **Cross-reference content claims:**
   - Check that feature lists in setup audit checks align with what bootstrap actually generates
   - Check that any "best practice" advice in one skill doesn't contradict the other

5. **Report discrepancies:**
   - For each issue, show: file, line number, what's inconsistent, and the correct value from the source of truth
   - Group by severity: `MISMATCH` (wrong value) vs `STALE` (outdated but not wrong) vs `MISSING` (should be present)
   - If no issues found, report: "Skills are in sync with core guides."
