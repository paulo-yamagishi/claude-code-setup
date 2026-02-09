# Sandboxing

Claude Code's sandbox restricts file system and network access, reducing approval fatigue and enabling safer autonomous operation.

---

## Why Sandbox

Without sandboxing, every potentially risky tool call requires user approval. The sandbox creates safe boundaries so Claude can operate autonomously within them:

- **Reduce approval prompts** — commands that stay within sandbox boundaries are auto-approved
- **Protect against prompt injection** — malicious instructions in code or dependencies cannot reach outside the sandbox
- **Limit blast radius** — mistakes are contained to the project directory

---

## Filesystem Isolation

The sandbox enforces three access tiers:

| Tier | Access | Scope |
|------|--------|-------|
| **Read/Write** | Full access | Current working directory and subdirectories |
| **Read-only** | Can read, cannot write | Other directories on the system |
| **Blocked** | No access | Sensitive system directories |

Claude can freely create, edit, and delete files within the project directory. It can read configuration files and references elsewhere, but cannot modify them.

---

## Network Isolation

Network access uses a domain-based proxy:

- **Allowed domains** — requests pass through without prompts
- **New domains** — Claude asks for user confirmation on first access
- **Blocked domains** — requests are denied

Once a domain is approved in a session, subsequent requests to it proceed without further prompts.

---

## OS-Level Enforcement

The sandbox uses operating system primitives, not just application-level checks.

### macOS

Uses **Seatbelt** (the same sandbox framework used by App Store apps) to enforce filesystem and network restrictions at the kernel level.

### Linux

Uses **bubblewrap** (`bwrap`) to create a restricted namespace with controlled mounts and network access.

### Windows (WSL2)

Uses the same Linux bubblewrap approach within the WSL2 environment.

---

## Enabling the Sandbox

### In-session

```
/sandbox
```

### At startup

```bash
claude --sandbox
```

---

## Modes

### Auto-allow mode

When sandboxing is active, commands that stay within sandbox boundaries are auto-approved without user confirmation. This is the key benefit — the sandbox replaces per-command approval with boundary-based approval.

### Regular permissions

Without sandboxing, Claude Code uses its standard permission system. Each tool call is evaluated against `allowedTools` and may prompt for approval.

The sandbox and permissions work together — you can use both for layered security.

---

## Configuration

Configure sandbox behavior in `.claude/settings.json`:

```json
{
  "sandbox": {
    "enabled": true
  }
}
```

---

## Security Benefits

### Prompt injection protection

If a dependency or file contains malicious instructions (e.g., "delete all files"), the sandbox prevents those operations from affecting anything outside the project.

### Malicious dependency protection

When running `npm install` or similar, the sandbox prevents post-install scripts from accessing sensitive files or making unauthorized network requests.

### Social engineering resistance

Even if Claude is tricked into running a harmful command, the sandbox limits the damage to the project directory.

---

## Limitations

- **Performance overhead** — sandbox enforcement adds a small latency to each operation
- **Compatibility** — some tools or workflows may not work within sandbox restrictions
- **Platform support** — requires macOS (Seatbelt) or Linux (bubblewrap); native Windows is not supported

---

## Best Practices

1. **Start restrictive** — enable sandboxing by default, loosen only when needed
2. **Combine with permissions** — use `allowedTools` alongside the sandbox for defense in depth
3. **Use for CI/headless** — sandboxing is especially valuable when running Claude unattended
4. **Test your workflow** — verify that your build tools and test runners work within sandbox restrictions before relying on it for production use
