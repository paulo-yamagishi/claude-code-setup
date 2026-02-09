# Planning Prompt Patterns

## Feature Planning

```
I need to implement [feature]. Before writing any code:

1. Explore the existing codebase to understand the current architecture
2. Identify all files that will need changes
3. Consider edge cases and error handling
4. Plan the implementation in ordered steps
5. Present the plan for my approval before proceeding
```

## Architecture Decision

```
I'm considering [approach A] vs [approach B] for [problem].

Analyze our codebase and recommend which approach fits better. Consider:
- Existing patterns and conventions in the code
- Maintenance burden
- Performance implications
- Testing complexity
```

## Task Breakdown

```
I need to [large task]. Break this into small, independently testable steps.
Each step should:
- Be completable in one session
- Have a clear verification method
- Not break existing functionality
```

## Estimation & Scoping

```
Before implementing [feature], help me understand the scope:
- Which files need changes?
- What are the dependencies?
- What tests need to be written or updated?
- Are there any risks or unknowns?
```

## Tips

- Always use `EnterPlanMode` for non-trivial tasks — it forces exploration before coding
- Let Claude explore the codebase first; don't prescribe the solution
- Review the plan before approving — Claude will show you the files it intends to change
- For very large tasks, plan in one session and execute in focused sub-sessions
