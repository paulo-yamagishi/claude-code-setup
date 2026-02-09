# Terminal and Environment Setup

Your terminal environment directly affects your Claude Code experience. A well-configured setup reduces friction and makes parallel workflows practical.

---

## Terminal Recommendations

### Ghostty

[Ghostty](https://ghostty.org) is purpose-built for modern CLI tools and pairs well with Claude Code:

- **Synchronized rendering** — no flickering during rapid output
- **24-bit color and Unicode** — full-fidelity display of diffs, diagrams, and status indicators
- **GPU-accelerated** — handles high-throughput output without lag
- **Native macOS/Linux** — no Electron overhead

### iTerm2

A solid alternative on macOS with mature features:

- Split panes for running Claude in one pane and tests in another
- Built-in search across scrollback
- Profile support for different project configurations

### Terminal.app / Windows Terminal

The defaults work fine. Upgrade if you notice rendering issues with Claude Code's output or need features like split panes.

---

## Status Line

Enable the Claude Code status line to keep tabs on context usage and git state without running commands:

```
/statusline
```

The status line shows:
- Current context window usage (percentage)
- Active git branch
- Session duration

This removes the need to manually run `/context` or `git branch` to check state.

---

## Tab and Window Organization

### Color-coded tabs

Assign distinct tab colors to different worktrees or task types:

- Green tab → main branch / production fixes
- Blue tab → feature branch
- Yellow tab → exploratory / read-only analysis

Most terminals (Ghostty, iTerm2, Windows Terminal) support per-tab or per-profile color schemes.

### Naming convention

Name tabs by worktree or task to avoid confusion when running 3-5 parallel sessions:

```
[feature-auth] claude    [bugfix-api] claude    [analysis] claude
```

### tmux layout

For terminal multiplexer users, a dedicated Claude Code layout keeps things organized:

```bash
# Create a tmux session with named windows per worktree
tmux new-session -s project -n main
tmux new-window -n feature
tmux new-window -n bugfix

# In each window, cd to the appropriate worktree and launch claude
```

---

## Voice Dictation

macOS has built-in voice dictation — press `fn` twice to activate. This is roughly 3x faster than typing for:

- Dictating long specifications and requirements
- Describing complex business logic
- Providing detailed bug reports with reproduction steps
- Writing CLAUDE.md entries and documentation

Voice dictation works directly in the terminal. Dictate your prompt, clean up any transcription errors, and submit. See `core/prompting-guide.md` for specification-driven prompting patterns that pair well with dictation.

---

## Data and Analytics CLI

For data-heavy workflows, integrate CLI tools so Claude can query data directly instead of context-switching to a browser.

### BigQuery (bq CLI)

```bash
# Install via gcloud SDK
gcloud components install bq

# Claude can then run queries directly
bq query --use_legacy_sql=false 'SELECT count(*) FROM dataset.table WHERE date > "2025-01-01"'
```

### Database CLIs

Connect Claude to databases via CLI tools or MCP servers:

| Approach | Example | Best For |
|---|---|---|
| CLI tools | `psql`, `mysql`, `sqlite3` | Simple queries, quick lookups |
| MCP servers | Postgres MCP, MySQL MCP | Structured access with schema awareness |
| API endpoints | REST/GraphQL data APIs | When direct DB access is restricted |

The pattern: instead of writing SQL yourself, describe what you need and let Claude write and execute the query.

```
How many users signed up last week? Break it down by referral source.
```

Claude writes the query, runs it via the appropriate CLI or MCP tool, and presents the results. See `core/mcp-guide.md` for setting up database MCP servers.

---

## Cross-References

- `core/session-management.md` — Git worktrees for parallel terminal sessions
- `core/mcp-guide.md` — MCP servers for database and API access
- `core/prompting-guide.md` — Voice dictation and specification-driven prompting
