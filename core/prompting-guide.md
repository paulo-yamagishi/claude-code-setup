# Prompting Techniques for Claude Code

Effective prompting is the highest-leverage skill for Claude Code. The difference between a mediocre and excellent result often comes down to how you frame the request.

---

## Challenge Prompts

Push Claude to validate its own work instead of accepting the first solution.

```
Grill me on these changes — what could go wrong in production?
```

```
Prove this works. Write a test for every edge case you can think of.
```

```
Don't make a PR until I can pass your test. Quiz me on what this code does.
```

Challenge prompts are especially useful after a complex implementation. They force Claude to think adversarially about its own output.

---

## Reset Prompts

When you're going down the wrong path, explicitly tell Claude to start over rather than iterating on a bad approach.

```
Scrap this and implement the elegant solution. What we have is too complex.
```

```
Step back. What's the simplest approach that solves the actual problem?
```

```
We're overengineering this. Start fresh with the minimum viable change.
```

If you've gone back and forth more than twice on the same approach, a reset prompt is almost always faster than continuing to iterate. See `core/workflow.md` for using plan mode to reset direction.

---

## Specification-Driven Prompting

The more detail you provide upfront, the fewer correction cycles you need. Write a short spec before handing off complex work.

```
Implement a rate limiter for the /api/search endpoint:
- Token bucket algorithm, 100 requests per minute per API key
- Return 429 with Retry-After header when exceeded
- Store state in Redis using the existing connection pool
- Add middleware to src/middleware/rateLimiter.ts
- Unit tests for bucket refill logic and edge cases
```

Contrast with the vague alternative: "add rate limiting to search." The spec eliminates ambiguity about algorithm, limits, storage, file placement, and test expectations.

### Writing specs with voice dictation

macOS voice dictation (press `fn` twice) is roughly 3x faster than typing. Use it for dictating long specs — you can always clean up formatting afterward. This is especially useful for describing complex business logic or multi-step workflows.

---

## Verification Prompts

Ask Claude to review its own work the way a teammate would.

```
Walk me through this change like a PR review. What would a senior engineer flag?
```

```
Diff the behavior between main and this branch. What changed from the user's perspective?
```

```
Run through the happy path, error path, and edge cases manually. Does this hold up?
```

Combine verification prompts with `core/workflow.md` plan mode — use plan mode not just for building, but for verification steps too.

---

## Learning Prompts

Use Claude Code as a teaching tool, not just a code generator.

```
Explain this module to me like I'm onboarding. What are the key abstractions and why were they chosen?
```

```
Create an ASCII diagram of how a request flows through this system.
```

```
Build me an HTML presentation explaining how our auth system works. Include diagrams.
```

### Spaced-repetition learning

Treat Claude as a study partner for unfamiliar codebases:

1. Ask Claude to explain a module
2. Work on something else for a day
3. Come back and ask Claude to quiz you on it
4. Repeat until the concepts stick

### Configuring output style

Use `/config` to set explanatory output preferences. Setting the output style to "Explanatory/Learning" makes Claude include more context about *why* it's making changes, which accelerates learning on unfamiliar codebases.

See `core/skills-guide.md` for creating reusable learning skills.

---

## Prompting Anti-Patterns

| Anti-Pattern | Why It Fails | Better Approach |
|---|---|---|
| "Make it better" | No criteria for "better" | Specify what to improve and how to measure it |
| "Fix all the bugs" | Unbounded scope | Point at a specific bug or error message |
| Iterating 5+ times on the same fix | Sunk cost; the approach may be wrong | Use a reset prompt and start fresh |
| Pasting 500 lines of context | Wastes context window | Let Claude read files with the Read tool |
| "Do everything in one shot" | Complex tasks need phases | Break into explore, plan, execute phases |

---

## Combining Techniques

The best results come from combining prompting patterns with workflow phases:

1. **Spec** — Write a detailed spec (specification-driven)
2. **Plan** — Ask Claude to plan the implementation (`/plan`)
3. **Challenge** — Spin up a second session and ask it to critique the plan
4. **Execute** — Approve and implement
5. **Verify** — Use verification prompts to review the result
6. **Learn** — Ask Claude to explain what it built and why

Cross-references:
- `core/workflow.md` — Plan mode and the explore/plan/execute/commit workflow
- `core/skills-guide.md` — Package prompting patterns as reusable skills
- `core/session-management.md` — Context management for long prompting sessions
