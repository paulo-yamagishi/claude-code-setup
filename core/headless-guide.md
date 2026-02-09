# Headless / Programmatic Mode

Claude Code can run non-interactively for CI/CD pipelines, scripts, and programmatic integrations using the `-p` flag.

## Basic Usage

```bash
claude -p "Explain what this project does"
```

This runs a single prompt and exits. Input can also be piped:

```bash
cat README.md | claude -p "Summarize this file"
```

## Output Formats

Control output with `--output-format`:

| Format | Description |
|--------|-------------|
| `text` | Plain text (default) — just the response content |
| `json` | Structured JSON with `session_id`, metadata, and result |
| `stream-json` | Newline-delimited JSON objects for real-time streaming |

### Text Output

```bash
claude -p "What does main.py do?" --output-format text
```

Returns only the response body as plain text.

### JSON Output

```bash
claude -p "List all TODO comments" --output-format json
```

Returns structured data:

```json
{
  "session_id": "abc-123",
  "result": "Found 3 TODO comments:\n- main.py:12 ...",
  "metadata": {
    "model": "claude-sonnet-4-5-20250929",
    "tokens_used": 1523
  }
}
```

### Streaming JSON

```bash
claude -p "Refactor utils.py" --output-format stream-json
```

Emits newline-delimited JSON objects as they are produced — useful for progress tracking in CI.

## Structured Output with JSON Schema

Force responses to match a specific schema:

```bash
claude -p "Analyze this codebase for security issues" \
  --output-format json \
  --json-schema '{"type":"object","properties":{"issues":{"type":"array","items":{"type":"object","properties":{"file":{"type":"string"},"severity":{"type":"string"},"description":{"type":"string"}},"required":["file","severity","description"]}}},"required":["issues"]}'
```

## Tool Permissions

Auto-approve specific tools to avoid interactive prompts:

```bash
claude -p "Fix the failing test" --allowedTools "Read,Edit,Bash"
```

Without this flag, tool calls that require approval will fail in headless mode.

## System Prompt Customization

### Append to Default Prompt

```bash
claude -p "Review this PR" --append-system-prompt "Focus on security issues only"
```

### Replace Entire System Prompt

```bash
claude -p "Review this PR" --system-prompt "You are a security auditor..."
```

## Continuing Conversations

### Resume Most Recent

```bash
claude -p "Now add error handling" --continue
```

### Resume Specific Session

```bash
claude -p "What about the edge cases?" --resume abc-123
```

The session ID is available from JSON output of previous runs.

## CI/CD Patterns

### Automated Commit Messages

```bash
claude -p "Generate a commit message for the staged changes" \
  --allowedTools "Bash" \
  --output-format text | git commit -F -
```

### Code Review in PRs

```bash
DIFF=$(gh pr diff $PR_NUMBER)
claude -p "Review this PR diff for issues: $DIFF" \
  --append-system-prompt "Output as GitHub-flavored markdown" \
  --output-format text
```

### Fan-out Across Files

```bash
for file in src/*.py; do
  claude -p "Add type hints to $file" \
    --allowedTools "Read,Edit" &
done
wait
```

### Pre-commit Validation

```bash
claude -p "Check staged files for common issues" \
  --allowedTools "Bash,Read" \
  --output-format json
```

## Claude Code SDK

For full programmatic control, use the official SDK packages:

### Python

```bash
pip install claude-code-sdk
```

```python
import claude_code_sdk

async def main():
    result = await claude_code_sdk.query(
        prompt="Explain this function",
        options={
            "allowedTools": ["Read", "Bash"],
        }
    )
    print(result.text)
```

### TypeScript

```bash
npm install @anthropic-ai/claude-code-sdk
```

```typescript
import { query } from "@anthropic-ai/claude-code-sdk";

const result = await query({
  prompt: "Explain this function",
  options: {
    allowedTools: ["Read", "Bash"],
  },
});
console.log(result.text);
```

The SDK provides the same capabilities as the CLI with native language integration, event streaming, and session management.
