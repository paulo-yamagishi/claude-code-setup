# Output Format — Setup Advisor

Reproduce the format below exactly. Read each section carefully.

## Layer 0: Health Banner

╔═══════════════════════════════════════════════╗
║                                               ║
║   SETUP HEALTH: 7.2 / 10 — Good              ║
║   Solid foundation, room to optimize.         ║
║                                               ║
╚═══════════════════════════════════════════════╝

> 🚀 **Biggest wins:** auto-format hooks (saves manual formatting
> on every edit) and deny rules (blocks dangerous commands).

- Use ╔═╗╚═╝║ box-drawing characters (double-line)
- Calculate the exact score BEFORE writing the banner. Never revise mid-output.
- "Biggest wins" names top 2 MISSING/IMPROVE categories by weight
  with concrete consequences

## Layer 1: Scorecard

Show ALL 11 rows. Use emoji status indicators.

| Category       | Status | Impact   | Notes                                                    |
|----------------|--------|----------|----------------------------------------------------------|
| CLAUDE.md      | ✅ GOOD    | HIGH   | Has commands, architecture, and style sections           |
| Settings       | ⚠️ IMPROVE | HIGH   | Missing deny rules — risky commands not blocked          |
| Hooks          | ❌ MISSING | HIGH   | No auto-format — you'll re-format manually on edit       |
| Skills         | ❌ MISSING | MEDIUM | No reusable workflows — repeating steps each time        |
| Rules          | ✅ GOOD    | MEDIUM | Modular rule files, well-organized                       |
| Agents         | ❌ MISSING | LOW    | No custom agents — missing code-reviewer, test-writer    |
| MCP            | ❌ MISSING | LOW    | No MCP servers — manual steps for GitHub, DB operations  |
| Plugins        | ❌ MISSING | LOW    | No plugins — missing code intelligence for typed langs   |
| Sandboxing     | ❌ MISSING | LOW    | Not sandboxed — extra permission prompts on every action |
| Model Config   | ❌ MISSING | LOW    | Using default model — no opusplan or effort tuning       |
| Output Styles  | ❌ MISSING | LOW    | No custom styles — inconsistent output across sessions   |

**Grade: 3.5 / 10 — Basic**

Rules:
- Status indicators: ✅ GOOD, ⚠️ IMPROVE, ❌ MISSING
- Impact column: HIGH (1.5 weight), MEDIUM (1.0), LOW (0.5)
- Notes must describe the *consequence*, not just state a fact
- Always show all 11 rows, never skip categories

## Layer 2: Suggestions

Numbered, sorted by impact (HIGH first, then MEDIUM, then LOW).
Blank line between each suggestion. Each has a "Why" line.

───────────────────────────────────────────
  📋 Suggestions (sorted by impact)
───────────────────────────────────────────

1. **[Hooks · HIGH]** Add auto-format hook for Prettier
   Why: Every Write/Edit triggers a format prompt. This hook eliminates it — saves ~30s per file touch.

2. **[Settings · HIGH]** Add deny rules for destructive commands
   Why: Without deny rules, `rm -rf /`, `DROP TABLE`, `git push --force` are one accidental approval away.

3. **[Skills · MEDIUM]** Create fix-issue skill for GitHub workflow
   Why: A /fix-issue skill turns `gh issue view → fix → test → commit` into one command.

4. **[Agents · LOW]** Add code-reviewer agent (sonnet)
   Why: Automated review catches style/logic issues before PR — no context switching to review manually.

5. **[MCP · LOW]** Configure GitHub MCP server
   Why: Without it, every `gh` call needs manual approval. MCP server gives Claude direct GitHub access.

───────────────────────────────────────────
  🔗 Recommended Workflows
───────────────────────────────────────────

6. **[Workflow · MEDIUM]** GitHub Issue → PR: end-to-end issue fixing
   Why: Combines /fix-issue skill + GitHub MCP + auto-format hook into a single flow — issue to merged PR in one session.

Rules:
- HIGH suggestions first, then MEDIUM, then LOW
- "Why" lines must be concrete (time saved, risk avoided), never generic
- Blank line between each numbered item
- Workflow suggestions come after individual suggestions, under their own separator
- Number workflows as continuation of the suggestion list (not restarting at 1)

## Layer 3: Interactive Prompt

End every report with:

**Say 'details N' for config snippet, or 'bootstrap' to apply all suggestions automatically.**

Replace N with actual suggestion numbers from your report.

When the user says "bootstrap", tell them to run `/bootstrap` — it has write access and will apply the configuration changes this audit recommends.
