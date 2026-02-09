# Code Review Prompt Patterns

## General Review

```
Review the changes in this PR/branch for:
- Correctness and logic errors
- Security vulnerabilities
- Performance issues
- Test coverage
- Code style consistency

Focus on substantive issues, not nitpicks.
```

## Security-Focused Review

```
Review this code with a security focus:
- Input validation and sanitization
- Authentication and authorization checks
- SQL injection, XSS, command injection risks
- Sensitive data exposure (logs, error messages, responses)
- Dependency vulnerabilities
```

## Pre-Merge Checklist

```
Before this PR merges, verify:
1. All tests pass
2. No regressions in existing functionality
3. Error handling covers edge cases
4. No hardcoded secrets or credentials
5. Changes are documented if they affect the public API
```

## Using the Code Reviewer Subagent

Launch a dedicated review agent to keep your main context clean:

```
Use the code-reviewer agent to review the changes on this branch compared to main.
```

This spawns an isolated agent that reads the diff and returns findings without cluttering your working session.

## Tips

- Review diffs, not entire files — focus on what changed
- Run tests as part of the review, don't just read the code
- For large PRs, review file-by-file with /clear between groups
