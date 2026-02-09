# Multi-Agent Orchestration with Agent Teams

## What Are Agent Teams?

Agent teams are an experimental feature for coordinating multiple Claude instances to tackle complex tasks collaboratively. Instead of a single agent doing everything sequentially, a team lead coordinates teammates that operate in parallel on a shared task list.

## Enabling Agent Teams

Set the environment variable:

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

## Architecture

Agent teams use a structured coordination model:

- **Team lead** -- The primary Claude instance that coordinates the work.
- **Teammates** -- Additional Claude instances that execute tasks.
- **Shared task list** -- A common queue of tasks with dependencies and status tracking. View with `Ctrl+T`.
- **Mailbox** -- Inter-agent messaging system for direct communication, broadcasting, and coordination.

## Display Modes (teammateMode)

Control how teammates are displayed with the `teammateMode` setting:

| Mode | Description |
|---|---|
| `auto` | Automatically choose the best display mode |
| `in-process` | Teammates run in the same terminal. Use `Shift+Up`/`Shift+Down` to select between them. |
| `tmux` | Teammates run in separate tmux split panes for full visibility. |

Configure in settings:

```json
{
  "teammateMode": "in-process"
}
```

## Task Management

### Task Dependencies

Tasks support dependencies with automatic unblocking. When a task completes, downstream tasks that were blocked on it are automatically unblocked and become available for teammates to claim.

### File-Locked Task Claiming

Tasks are claimed with file locks to prevent multiple teammates from working on the same task simultaneously.

### Plan Approval

The team lead can require plan approval before teammates begin execution, ensuring alignment on approach.

## Inter-Agent Communication

### Delegate Mode

Press `Shift+Tab` to enter delegate mode, allowing you to directly assign work to specific teammates.

### Direct Messaging

Teammates can send messages directly to specific other teammates via the mailbox system.

### Broadcasting

Send messages to all teammates simultaneously for coordination announcements or shared context.

## Hooks for Agent Teams

Two hook events are specific to agent teams:

| Event | When | Can Block? |
|---|---|---|
| `TeammateIdle` | A teammate is about to go idle | Yes (exit 2 only) |
| `TaskCompleted` | A task is marked complete | Yes (exit 2 only) |

These hooks enable custom logic when teammates finish work or when tasks reach completion.

## Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl+T` | View the shared task list |
| `Shift+Tab` | Enter delegate mode |
| `Shift+Up` / `Shift+Down` | Select between teammates (in-process mode) |

## Best Practices

**Keep team size small.** Two to four teammates is usually sufficient. More agents means more coordination overhead and more chances for inconsistency.

**Define clear responsibilities.** Each teammate should have a distinct role. Overlapping responsibilities lead to duplicated or conflicting work.

**Use task dependencies to manage ordering.** Do not rely on timing or hope that agents finish in the right order. Explicitly declare what blocks what.

**The team lead should coordinate, not duplicate.** The lead's job is to break down work, delegate, and combine results -- not to redo what teammates already did.

**Start simple.** Begin with a single subagent before graduating to multi-agent patterns. Add complexity only when the task genuinely requires it.

## Limitations

- **Experimental feature.** Enable with `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`. The API and behavior may change.
- **No shared context.** Agents communicate through the mailbox and task list, not shared conversation history.
- **Coordination overhead.** Spawning and managing multiple agents takes time and tokens. For simple tasks, a single agent is faster.
- **Conflict potential.** Multiple agents editing the same files can create conflicts. Design your task decomposition to minimize file overlap.
- **Token cost.** Each agent consumes its own token budget. Multi-agent workflows cost more than single-agent approaches.
