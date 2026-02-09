# CLI Reference

## Installation

```bash
npm install -g @anthropic-ai/claude-code
```

Requires Node.js 18 or later.

## Basic Usage

```bash
claude [options] [prompt]
```

## Modes of Operation

### Interactive Mode

Run `claude` with no arguments to start an interactive session:

```bash
claude
```

You get a REPL where you type prompts and Claude responds. Use slash commands (listed below) to control the session.

### One-Shot Mode (Print Mode)

Run a single prompt and exit. Output is printed to stdout. Non-interactive -- suitable for scripts and automation.

```bash
claude -p "explain what this project does"
claude -p "find all TODO comments in the codebase"
```

### Pipe Mode

Pipe content into Claude for processing:

```bash
cat src/index.ts | claude -p "review this code"
git diff | claude -p "write a commit message for these changes"
echo "def fib(n):" | claude -p "complete this function"
```

## Key Flags

| Flag | Description |
|------|-------------|
| `-p, --prompt` | Run in non-interactive (print) mode |
| `--model` | Override the model (`sonnet`, `opus`, `haiku`, or full model name) |
| `--add-dir` | Add additional working directories |
| `--agent` | Specify the agent to use |
| `--agents` | Define subagents via JSON |
| `--allowedTools` | Tools to allow without prompting |
| `--disallowedTools` | Tools to deny |
| `--append-system-prompt` | Append text to the system prompt |
| `--system-prompt` | Replace the entire system prompt |
| `--permission-mode` | Set permission mode |
| `--dangerously-skip-permissions` | Skip all permission prompts (for CI/CD only) |
| `--json-schema` | Validated JSON output (print mode) |
| `--output-format` | Output format: `text`, `json`, `stream-json` |
| `--input-format` | Input format: `text`, `stream-json` |
| `--max-turns` | Limit agentic turns (print mode) |
| `--max-budget-usd` | Maximum spend in USD (print mode) |
| `--mcp-config` | MCP server configuration file |
| `--strict-mcp-config` | Strict MCP configuration (fail on errors) |
| `--continue` | Continue the most recent session |
| `--resume` | Resume a previous session by ID |
| `--fork-session` | Fork an existing session |
| `--from-pr` | Start from a pull request |
| `--tools` | Restrict available tools |
| `--debug` | Show debug output |
| `--verbose` | Show verbose output and tool call details |
| `--remote` | Create a web session |
| `--teleport` | Resume a web session locally |
| `--plugin-dir` | Load plugins from a directory |
| `--teammate-mode` | Set teammate display mode |
| `--fallback-model` | Fallback model if primary is unavailable |
| `--no-session-persistence` | Disable session persistence |
| `--session-id` | Use a specific session UUID |

### Examples

```bash
# Use a specific model
claude --model opus -p "refactor this function"

# Limit turns to prevent runaway agents
claude -p "fix the build" --max-turns 10

# Set a budget limit
claude -p "implement feature X" --max-budget-usd 5.00

# Get JSON output for scripting
claude -p "list all API endpoints" --output-format json

# Get validated JSON matching a schema
claude -p "extract metadata" --json-schema '{"type":"object","properties":{"title":{"type":"string"}}}'

# Resume a previous session
claude --resume

# Continue the last session
claude --continue

# Fork from a pull request
claude --from-pr 123

# Allow only read operations
claude --allowedTools "Read,Glob,Grep" -p "analyze the codebase"

# Deny shell access
claude --disallowedTools "Bash" -p "review src/auth.ts"

# Add additional directories
claude --add-dir /path/to/other/project

# Create a web session
claude --remote
```

## Slash Commands (Interactive Mode)

These commands are available inside an interactive Claude session:

| Command | Description |
|---------|-------------|
| `/help` | Show help and available commands |
| `/clear` | Clear conversation context and start fresh |
| `/compact` | Compact context by summarizing the conversation |
| `/context` | Show current context window usage |
| `/mcp` | Show MCP server connection status and OAuth flows |
| `/resume` | Resume a previous session |
| `/config` | Open Claude Code settings |
| `/quit` or `/exit` | Exit the session |

## Environment Variables

| Variable | Description |
|----------|-------------|
| `ANTHROPIC_API_KEY` | API key for authentication |
| `ANTHROPIC_AUTH_TOKEN` | Auth token (alternative to API key) |
| `ANTHROPIC_MODEL` | Default model to use |
| `CLAUDE_CODE_SUBAGENT_MODEL` | Model for subagents |
| `CLAUDE_CODE_EFFORT_LEVEL` | Reasoning effort: `low`, `medium`, `high` |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | Disable automatic memory |
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` | Enable agent teams (`1`) |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` | Max output tokens (default: 32000, max: 64000) |
| `MAX_MCP_OUTPUT_TOKENS` | Max MCP output tokens (default: 25000) |
| `BASH_DEFAULT_TIMEOUT_MS` | Default bash timeout in ms |
| `DISABLE_AUTOUPDATER` | Disable automatic updates |
| `DISABLE_ERROR_REPORTING` | Disable error reporting |
| `DISABLE_TELEMETRY` | Disable telemetry |

### Example: Setting Environment Variables

```bash
# In your shell profile (.bashrc, .zshrc)
export ANTHROPIC_API_KEY="sk-ant-..."
export ANTHROPIC_MODEL="claude-opus-4-6"
export CLAUDE_CODE_EFFORT_LEVEL="high"
```

## CI/CD Usage

For automation, combine `-p` with `--dangerously-skip-permissions` to run without interactive prompts:

```bash
claude -p "run the test suite and report results" \
  --dangerously-skip-permissions \
  --max-turns 10 \
  --output-format json
```

**Important**: `--dangerously-skip-permissions` disables all safety prompts. Only use this in sandboxed CI environments, never on machines with access to production systems or sensitive data.

### CI/CD Patterns

```bash
# Code review in CI
git diff main...HEAD | claude -p "review these changes" \
  --dangerously-skip-permissions \
  --output-format text

# Generate release notes
claude -p "generate release notes from commits since last tag" \
  --dangerously-skip-permissions \
  --max-turns 5

# Run and fix failing tests with budget limit
claude -p "run tests, if any fail fix them" \
  --dangerously-skip-permissions \
  --max-turns 15 \
  --max-budget-usd 2.00
```

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | Success -- task completed normally |
| `1` | Error -- task failed or Claude encountered an error |
| `2` | Permission denied -- a required permission was not granted |

Use exit codes in scripts to handle failures:

```bash
claude -p "run the linter" --dangerously-skip-permissions
if [ $? -ne 0 ]; then
  echo "Linter failed"
  exit 1
fi
```
