# Model Configuration

How to select models, configure effort levels, and tune Claude Code's model behavior.

---

## Model Aliases

Claude Code uses aliases instead of raw model IDs:

| Alias | Description |
|-------|-------------|
| `default` | The default model (currently Sonnet) |
| `sonnet` | Claude Sonnet — fast, cost-effective |
| `opus` | Claude Opus — highest capability |
| `haiku` | Claude Haiku — fastest, lowest cost |
| `opusplan` | Opus for planning, Sonnet for execution |
| `sonnet[1m]` | Sonnet with extended 1M token context |

---

## Setting the Model

### In-session

Use the `/model` command to switch models during a conversation:

```
/model opus
/model sonnet
/model opusplan
```

### At startup

```bash
claude --model opus
```

### Via environment variable

```bash
export ANTHROPIC_MODEL=opus
```

### Via settings file

In `.claude/settings.json`:

```json
{
  "model": "sonnet"
}
```

**Priority order:** `--model` flag > `ANTHROPIC_MODEL` env > settings file > default.

---

## Opus Plan Mode

The `opusplan` alias uses Opus for plan mode (thinking and planning) and Sonnet for act mode (code execution). This gives you Opus-quality reasoning at Sonnet-level cost for implementation.

```bash
claude --model opusplan
```

This is effective for complex tasks where planning quality matters but execution is straightforward.

---

## Effort Levels

Control how much thinking Claude does before responding.

| Level | Behavior |
|-------|----------|
| `low` | Minimal thinking, fastest responses |
| `medium` | Balanced (default) |
| `high` | Extended thinking, most thorough |

### Set effort in-session

Use the `/model` slider or command to adjust effort level.

### Via environment variable

```bash
export CLAUDE_CODE_EFFORT_LEVEL=high
```

### Via settings file

```json
{
  "effortLevel": "high"
}
```

Higher effort is useful for complex debugging, architecture decisions, and multi-file refactors. Lower effort suits quick edits and simple questions.

---

## Extended Context

Append `[1m]` to a model alias to enable the 1 million token context window:

```bash
claude --model sonnet[1m]
```

Use extended context for:
- Large codebases where many files need to be in context simultaneously
- Long sessions with extensive conversation history
- Tasks requiring analysis of many files at once

Extended context has higher per-token costs.

---

## Environment Variables for Model Override

Override which underlying model backs each alias:

| Variable | Purpose |
|----------|---------|
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | Model ID used when `opus` alias is selected |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | Model ID used when `sonnet` alias is selected |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | Model ID used when `haiku` alias is selected |
| `CLAUDE_CODE_SUBAGENT_MODEL` | Model used for subagent tasks |

```bash
# Use a specific model version for the sonnet alias
export ANTHROPIC_DEFAULT_SONNET_MODEL=claude-sonnet-4-5-20250929

# Use a cheaper model for subagents
export CLAUDE_CODE_SUBAGENT_MODEL=haiku
```

---

## Prompt Caching

Claude Code uses prompt caching to reduce costs and latency for repeated context. To disable:

```bash
export DISABLE_PROMPT_CACHING=1
```

Per-model cache control is available through model-specific environment variables. Check the current documentation for the latest cache configuration options.

---

## Fast Mode

Fast mode trades higher cost for lower latency by using a more capable model.

### Toggle in-session

```
/fast
```

### Via settings file

```json
{
  "fastMode": true
}
```

When fast mode is enabled and the primary model hits a rate limit, Claude Code automatically falls back to the standard model rather than failing.
