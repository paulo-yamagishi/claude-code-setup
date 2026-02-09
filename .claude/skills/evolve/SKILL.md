---
name: evolve
description: Auto-maintain Claude Code config files as your codebase evolves
allowed-tools:
  - Read
  - Edit
  - Glob
  - Grep
  - Bash(git diff *)
  - Bash(git log *)
---

# /evolve — Auto-Maintain Claude Code Configuration

Automatically keep Claude Code config files accurate as a codebase evolves. Run after completing a task or wire into a Stop hook for hands-free maintenance.

## Phase 0 — Self-Install (with consent)

Check whether the Stop hook is registered, and ask the user before installing:

1. Read `.claude/settings.json` (if missing or empty, start with `{}`)
2. Check if any existing Stop hook prompt already contains `"evolve"` or `"config maintenance"`
3. **If already present:** continue silently to Phase 1
4. **If not found:** show the user the hook JSON that would be added:
   ```json
   {
     "type": "prompt",
     "prompt": "Before finishing, silently evaluate if any Claude Code configuration files need updating based on the work just completed. Check CLAUDE.md accuracy, CHANGELOG.md Unreleased entries, skill/rule/agent validity, and settings alignment. Only edit files that are actually stale. Report changes in one line or stay silent."
   }
   ```
5. **Ask the user:** "Install Stop hook for automatic config maintenance? (y/n)"
6. **If confirmed:** merge the hook into settings and save:
   - If `hooks` key is missing, create `{"hooks": {"Stop": [{"hooks": [<entry>]}]}}`
   - If `hooks.Stop` exists, append the new hook entry to the existing array's `hooks` list
   - Preserve all other keys in the settings file
   - Report: `[evolve] Installed: Stop hook added for automatic config maintenance`
7. **If declined:** report `[evolve] Skipped: Stop hook not installed. Run /evolve manually when needed.` and continue to Phase 1

## Phase 1 — Detect What Changed

Gather recent changes to understand what work was done:

```bash
git diff --name-only
git diff --staged --name-only
git log --oneline -5
```

Categorize every changed file into: **source code**, **tests**, **configs**, **dependencies**, **docs**, **new files/dirs**.

**Exit silently if:**
- No meaningful changes detected
- ALL changed files are under `.claude/` (avoid self-referential loops)

## Phase 2 — Evaluate Each Config File

For each Claude Code config file that exists in the project, check whether it is stale:

| Config File | What to Check |
|---|---|
| `CLAUDE.md` | Structure section matches actual dirs; Commands section has correct test/lint/build commands; Architecture reflects current state; any new key conventions emerging |
| `CHANGELOG.md` | Unreleased section has entry for work just done; skip if no CHANGELOG exists |
| `.claude/skills/*/SKILL.md` | Commands referenced still work; file paths still valid; any repeated workflow in this session that deserves a new skill |
| `.claude/rules/*.md` | Conventions still match actual code patterns; scoped paths still valid |
| `.claude/agents/*.md` | Tool lists still valid; model choices still appropriate |
| `.claude/settings.json` | New commands used that should be in allow list; formatter/linter commands still correct in hooks |
| `.mcp.json` | Existing config still valid (don't auto-add servers) |

Only flag files where the content is actually inconsistent with the current codebase state.

## Phase 3 — Apply Minimal Updates

- **Only edit files that are actually stale** — skip everything else
- **Use targeted edits** — never rewrite an entire file
- **Respect existing style** — match the formatting of the file being edited
- **New skills:** create only if a clear repeatable pattern was found (at least 2 similar operations in the current session)
- **CHANGELOG:** add a concise one-line entry under the appropriate subsection in `[Unreleased]`

## Phase 4 — Report

- **Changes made:** `[evolve] Updated: CLAUDE.md (new dir documented), CHANGELOG.md (added feature entry)`
- **New skill suggested:** `[evolve] Suggestion: consider creating /deploy skill for the deployment workflow used today`
- **Nothing needed:** complete silence — no output at all

## Rules

1. **NEVER commit** — only edit files, let the user decide when to commit
2. **NEVER create files without clear justification** — threshold: repeatable pattern seen at least 2 times in the session
3. **Skip if only `.claude/` files changed** — avoids self-referential loops
4. **Conservative** — when uncertain, suggest instead of edit
5. **Fast** — no sub-skill invocations, no web fetches, just local file reads and edits
6. **Respect existing style** — match formatting of the file being edited
7. **Project-agnostic** — no hardcoded paths, languages, or tools; all detection-based

## Stop Hook

The `/evolve` skill **offers to install its Stop hook** on first run (Phase 0). You will be asked for consent before any settings are modified.

To remove it, delete the Stop hook entry containing `"config maintenance"` from `.claude/settings.json`.
