# Checkpointing

Claude Code automatically tracks file edits made during each prompt, allowing you to rewind your project to any previous state. Checkpoints persist across sessions and are cleaned up after 30 days by default.

## How It Works

Every time Claude edits a file in response to a prompt, a checkpoint is created. This captures:
- The state of all modified files before and after the edit
- The conversation context at that point

Checkpoints are stored locally and persist across sessions. The default retention period is 30 days.

## Rewinding

### Opening the Rewind Menu

- Press **Esc** twice (`Esc+Esc`) to open the rewind interface
- Or use the `/rewind` command

The menu shows a timeline of checkpoints with file changes summarized at each step.

### Rewind Actions

| Action | Effect |
|--------|--------|
| **Restore code + conversation** | Reverts both files and conversation to the checkpoint state |
| **Restore conversation only** | Resets conversation context but keeps current file state |
| **Restore code only** | Reverts files but keeps the current conversation |
| **Summarize from here** | Compresses conversation from this point forward, freeing context window space |

### Restore vs Summarize

**Restore** reverts state — files, conversation, or both go back to exactly how they were at that checkpoint.

**Summarize** keeps the current state intact but compresses the conversation history from the selected checkpoint onward. Earlier context remains untouched. This frees up context window space without losing your current work.

## Use Cases

### Exploring Alternatives

Try an approach, checkpoint, then rewind and try a different one:

```
You: "Implement auth using JWT"
  → Claude makes changes (checkpoint created)
You: [Esc+Esc → Restore code + conversation]
You: "Implement auth using session cookies instead"
```

### Recovering from Mistakes

If Claude makes changes that break your project:

```
You: [Esc+Esc → Restore code only]
```

Files revert while keeping conversation context so Claude understands what went wrong.

### Iterating on Features

Build incrementally with safe rollback points:

```
You: "Add basic user model"         → checkpoint 1
You: "Add validation"               → checkpoint 2
You: "Add email notifications"      → checkpoint 3
You: [Esc+Esc → Restore to checkpoint 2]
You: "Add SMS notifications instead"
```

### Freeing Context Window

During long sessions, summarize older checkpoints to reclaim context space:

```
You: [Esc+Esc → select early checkpoint → Summarize from here]
```

This compresses older conversation turns while preserving their key information.

## Limitations

- **Bash commands are NOT tracked** — changes made by shell commands (file moves, database modifications, `rm`, etc.) are not captured in checkpoints
- **External changes are not tracked** — edits made outside Claude Code (in your editor, by other tools) are not part of the checkpoint history
- **Not a replacement for git** — checkpoints are local, temporary, and lack branching, merging, or remote backup. Always commit important work to version control
