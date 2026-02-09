# Claude Code Setup

Three skills that form a complete lifecycle for Claude Code configuration:
**audit** → **configure** → **maintain**.

![Version](https://img.shields.io/badge/version-1.0.0-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Claude Code](https://img.shields.io/badge/Claude_Code-compatible-8A2BE2)

## The Problem

Claude Code has 11 configuration surfaces — CLAUDE.md, settings, hooks, skills, agents, MCP, rules, plugins, sandboxing, model config, and output styles. Most projects use 2-3. The built-in `/init` only creates a basic `CLAUDE.md`. This toolkit **finds** what's missing, **sets it up**, and **keeps it accurate** as your code evolves.

## Requirements

- [Claude Code](https://code.claude.com/docs) installed:

```bash
# Native installer (recommended)
curl -fsSL https://claude.ai/install.sh | bash

# Or via Homebrew
brew install --cask claude-code
```

## Quick Start

```bash
# 1. Install the skills
mkdir -p ~/.claude/skills/{setup,bootstrap,evolve}
curl -o ~/.claude/skills/setup/SKILL.md \
  https://raw.githubusercontent.com/paulo-yamagishi/claude-code-setup/main/skill/SKILL.md
curl -o ~/.claude/skills/bootstrap/SKILL.md \
  https://raw.githubusercontent.com/paulo-yamagishi/claude-code-setup/main/.claude/skills/bootstrap/SKILL.md
curl -o ~/.claude/skills/evolve/SKILL.md \
  https://raw.githubusercontent.com/paulo-yamagishi/claude-code-setup/main/.claude/skills/evolve/SKILL.md
```

```bash
# 2. Run /setup in any project → see your config score
/setup
```

```bash
# 3. Say "bootstrap" → config generated for every gap
bootstrap
```

That's it. `/evolve` activates automatically via a Stop hook (with your consent) to keep config accurate over time.

---

## The Lifecycle

```
/setup                /bootstrap              /evolve
 audit ──────────▶   configure ──────────▶   maintain
                                               │
 • Scans 11 categories   • 9-phase flow        • Stop hook (auto)
 • Scores each one        • Greenfield support  • Detects drift
 • Suggests fixes         • Reads /setup report • Minimal edits
 • Hands off to bootstrap • Asks before writing • Never commits
```

Run `/setup` first to see the gaps. Say "bootstrap" to fill them. `/evolve` keeps everything accurate as your code changes — no manual upkeep.

---

## /setup — Audit Your Config

Type `/setup` in any project and Claude will:

1. **Audit** your current config across 11 categories
2. **Suggest** improvements tailored to your tech stack — individual configs plus complete workflows
3. **Apply** only what you approve — shows a scorecard first, details on request

### Usage

| Command | What it does |
|---------|-------------|
| `/setup` | Full audit across all 11 categories |
| `/setup quick` | Show only gaps and missing items |
| `/setup hooks` | Audit only hooks configuration |
| `/setup mcp` | Audit only MCP server setup |
| `/setup settings` | Audit only settings and permissions |
| `/setup skills` | Audit only skills |
| `/setup agents` | Audit only custom agents |
| `/setup claude-md` | Audit only CLAUDE.md quality |
| `/setup rules` | Audit only rules |
| `/setup plugins` | Audit only plugins |
| `/setup sandbox` | Audit only sandboxing |
| `/setup model` | Audit only model configuration |
| `/setup styles` | Audit only output styles |

### What Gets Audited

| Category | What's checked |
|----------|---------------|
| CLAUDE.md | Exists, under 500 lines, has key sections |
| Settings | Permissions configured, deny rules, local overrides |
| Hooks | Auto-format, notifications, dangerous command blocking |
| Skills | Exist, proper format, common skills (fix-issue, code-review) |
| Agents | Exist, tool restrictions, appropriate model selection |
| MCP | Configured, no hardcoded secrets, minimal server count |
| Rules | Exist, split by topic, focused files |
| Plugins | Installed, code intelligence for typed languages |
| Sandboxing | Enabled, auto-allow mode, allowed domains |
| Model Config | Model alias set, effort level configured |
| Output Styles | Custom styles directory, team styles |

### Example Output

```
## Setup Scorecard

| Category      | Status  | Notes                           |
|---------------|---------|----------------------------------|
| CLAUDE.md     | GOOD    | 120 lines, has key sections     |
| Settings      | IMPROVE | Missing deny rules              |
| Hooks         | MISSING | No hooks configured             |
| Skills        | MISSING | No skills directory             |
| MCP           | GOOD    | GitHub configured               |

Score: 2/11 GOOD · 1 IMPROVE · 8 MISSING

Say 'details 1' for config snippet, or 'bootstrap' to apply all suggestions.
```

---

## /bootstrap — Configure a Project

Type `/bootstrap` in any project — new or existing — and Claude will interactively configure everything.

### 9-Phase Flow

0. **Check for setup report** — uses `/setup` findings if available
1. **Detect** project stack (language, framework, package manager, tools)
2. **Greenfield scaffolding** — empty directory? Asks what to build, initializes the project
3. **Generate CLAUDE.md** — tailored to your stack
4. **Configure settings** — permissions, deny rules, hooks (auto-format, lint, notifications), sandbox, model config
5. **Create skills** — `/fix-issue`, `/code-review`, plus conditional ones per stack
6. **Create agents** — code-reviewer, test-writer, optional security-auditor
7. **Set up MCP** — GitHub, database, Slack based on detection
8. **Generate rules and output styles** — code style, testing, git conventions

### /setup → /bootstrap Handoff

Run `/setup` first, then say "bootstrap" in the same conversation:

```
/setup  ──audit──▶  scorecard + suggestions
                         │
  user says "bootstrap"  │
                         ▼
/bootstrap  ──▶  "Found your /setup audit — focusing on the gaps."
                 • Prioritizes IMPROVE and MISSING categories
                 • Skips prompting for GOOD categories
```

Running `/bootstrap` standalone works fine — it does full detection on its own.

### Usage

```
/bootstrap              # Full interactive setup
/bootstrap express      # Hint: Express.js project
/bootstrap fastapi      # Hint: FastAPI project
```

---

## /evolve — Auto-Maintain Config

`/evolve` keeps your Claude Code configuration accurate as your codebase changes. It runs after each task via a Stop hook — with your consent.

### What It Does

After each task completes, `/evolve` silently checks whether any config files have drifted:

| Config File | What's Checked |
|---|---|
| `CLAUDE.md` | Structure matches actual dirs, commands are correct |
| `CHANGELOG.md` | Unreleased section has entry for work just done |
| `.claude/skills/` | Commands and file paths still valid |
| `.claude/rules/` | Conventions still match actual code patterns |
| `.claude/agents/` | Tool lists and model choices still appropriate |
| `.claude/settings.json` | New commands that should be in allow list |

### How It Works

1. **First run:** asks permission to install a Stop hook in `.claude/settings.json`
2. **After each task:** detects what changed via `git diff`, evaluates each config file
3. **Applies minimal edits** — only files that are actually stale
4. **Reports changes** in one line, or stays completely silent if nothing needed

### Safety Guarantees

- **Never commits** — only edits files, you decide when to commit
- **Never creates files** unless a clear repeatable pattern was found (2+ times)
- **Skips `.claude/` loops** — exits silently if only config files changed
- **Conservative** — when uncertain, suggests instead of editing
- **Removable** — delete the Stop hook entry from `.claude/settings.json` to disable

---

## Detection Matrix

Languages and tools recognized by `/setup` and `/bootstrap`:

| Language | Frameworks | Formatter | Linter | Test Runner |
|----------|-----------|-----------|--------|-------------|
| Node.js / TypeScript | React, Vue, Next.js, Express, Fastify, Hono | Prettier, Biome | ESLint, Biome (flat config) | Jest, Vitest |
| Python | Django, Flask, FastAPI | Black, Ruff | Ruff, Pylint | pytest |
| Rust | Actix, Axum, Rocket | rustfmt | Clippy | cargo test |
| Go | Gin, Echo, Chi | gofmt | golangci-lint | go test |
| Java / JVM | Spring Boot, Quarkus | — | — | — |
| Ruby | Rails, Sinatra | — | — | — |
| PHP | Laravel, Symfony | PHP CS Fixer | — | PHPUnit |

Package managers: npm, yarn, pnpm, bun, pip, poetry, uv, cargo, go mod, bundle, composer.

---

## What's Included

### 3 Installable Skills

| Skill | Purpose |
|-------|---------|
| `/setup` | Audit existing config, score 11 categories, suggest fixes |
| `/bootstrap` | 9-phase interactive config generator (new + existing projects) |
| `/evolve` | Auto-maintain config via Stop hook after every task |

### 26 Core Reference Guides (`core/`)

Hooks, skills, MCP, settings, agents, model config, sandboxing, plugins, output styles, CLI reference, debugging, testing strategy, prompting, git conventions, session management, subagents, headless mode, checkpointing, terminal setup, workflows, and more.

### 6 Language-Specific Project Guides (`projects/`)

Python, TypeScript, Go, Rust, Fullstack, Data Science — each extends the core guides with stack-specific hooks, settings, and skills.

### 14 Copy-Paste Templates (`templates/`)

Ready-to-use templates for agents, skills, rules, settings, CLAUDE.md, MCP servers, and bootstrap scaffolding.

---

## FAQ

**How is this different from `/init`?**
`/init` creates a basic `CLAUDE.md`. This toolkit audits all 11 config surfaces, generates settings, hooks, skills, agents, MCP configs, rules, and output styles — then keeps them accurate over time.

**Can I use just one skill?**
Yes. Each skill works standalone. `/setup` only reads and reports. `/bootstrap` only writes with your approval. `/evolve` only runs if you install its hook.

**Will `/evolve` overwrite my config?**
No. It makes minimal targeted edits to stale files. It never rewrites entire files, never commits, and exits silently if nothing needs updating.

**What if `/bootstrap` runs on a project with existing config?**
It detects existing files and offers three choices per file: overwrite, merge, or skip. Nothing is lost without your approval.

---

## Contributing

Issues and PRs are welcome. If you find inaccurate config examples, missing Claude Code features, or detection gaps for your stack, please open an issue.

## License

[MIT](LICENSE)

## Official Docs

- [Claude Code Documentation](https://code.claude.com/docs)
