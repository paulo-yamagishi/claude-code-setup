---
name: research
description: Fetch official docs and community sources for a Claude Code topic, then integrate findings
argument-hint: "[topic]"
allowed-tools:
  - Read
  - Edit
  - Write
  - Glob
  - Grep
  - WebFetch
  - WebSearch
---

# Research

Research a Claude Code topic using official docs and community sources, then integrate verified findings.

## Steps

1. **Parse topic:** Use `$ARGUMENTS` as the research topic (e.g., "hooks", "plugins", "model configuration").

2. **Fetch official docs:**
   - WebFetch `https://docs.anthropic.com/en/docs/claude-code` and navigate to the relevant section
   - Extract current official information about the topic: features, config format, options, limitations

3. **Search community sources:**
   - WebSearch for: `"claude code" $ARGUMENTS site:reddit.com` — recent discussions and tips
   - WebSearch for: `"claude code" $ARGUMENTS` — blog posts, tutorials, announcements
   - Extract actionable tips, undocumented patterns, and real-world usage insights

4. **Read existing repo content:**
   - Grep for `$ARGUMENTS` across `core/`, `projects/`, `skill/`, `.claude/skills/` to find what we already cover
   - Read the most relevant files to understand current state of our documentation

5. **Cross-reference and classify findings into three sections:**

   **Confirmed new info** — Information verified against official docs that is not yet in our guides. Include source URL.

   **Community tips** — Plausible patterns from real usage not found in official docs. Mark each as `[UNVERIFIED]`. Include source URL.

   **Outdated content in our repo** — Anything in our guides that contradicts current official docs. Include file path and line number.

6. **Present report** to the user with the three sections above.

7. **Ask user** which findings to integrate, then update the relevant `core/` or `projects/` files accordingly.
