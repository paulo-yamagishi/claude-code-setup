# Systematic Debugging

## The Debugging Loop

Follow this sequence every time. Skipping steps leads to wasted time.

```
Reproduce → Isolate → Understand → Fix → Verify
```

1. **Reproduce**: Can you make the bug happen consistently? If not, gather more information before proceeding.
2. **Isolate**: Narrow down where the bug lives. Which file, which function, which line?
3. **Understand**: Why does the bug happen? What is the code doing vs. what should it do?
4. **Fix**: Make the smallest change that corrects the behavior.
5. **Verify**: Run tests. Confirm the bug is gone. Confirm nothing else broke.

## Using Claude for Debugging

Claude is effective at debugging because it can explore your codebase, form hypotheses, and test them — all in one session.

### Share Error Messages and Stack Traces

Paste the full error. Don't paraphrase it. Claude needs the exact text.

```
> I'm getting this error when I run the app:
>
> TypeError: Cannot read properties of undefined (reading 'map')
>     at UserList (src/components/UserList.tsx:14:22)
>     at renderWithHooks (node_modules/react-dom/...)
```

### Let Claude Explore

Claude will use Glob, Grep, and Read to investigate. A typical debugging session looks like:

```
> debug this error — the API returns 500 when creating a new project
```

Claude will:
- Read the route handler for project creation
- Check the database model and schema
- Look at recent changes to related files
- Read test files for expected behavior
- Grep for similar patterns elsewhere in the codebase
- Form a hypothesis and propose a fix

### Claude Checks Related Tests and Documentation

Claude doesn't just look at the broken code. It reads tests to understand intended behavior and checks documentation for API contracts and constraints.

## Common Pitfalls

### Stale Dependencies

After pulling changes, install dependencies before debugging:

```bash
# Node.js
npm install

# Python
pip install -r requirements.txt
# or
pip install -e .

# Go
go mod tidy
```

Half of "it works on their machine" problems are stale dependencies.

### Wrong Environment

Check the basics before digging into code:

```bash
node --version    # Expected v20+? Running v16?
python --version  # Expected 3.11? Running 3.8?
echo $DATABASE_URL  # Is it pointing at the right database?
```

Environment mismatches cause subtle bugs that look like code problems.

### Import Errors

- **Relative vs absolute paths**: `./utils` vs `src/utils` vs `@/utils`
- **Missing `__init__.py`**: Python won't recognize a directory as a package without it
- **Case sensitivity**: `UserList` vs `userList` — works on macOS, breaks on Linux
- **Circular imports**: A imports B, B imports A — restructure to break the cycle

### Type Errors

Types lie at boundaries. Check:
- API responses (the actual JSON vs. your TypeScript interface)
- Database query results (null vs. undefined vs. empty string)
- Environment variables (always strings, even `"true"` and `"3000"`)
- User input (never trust it, always validate)

### Race Conditions

Async code is where bugs hide:

```javascript
// Bug: response might not be set yet
fetchData();
console.log(response);

// Fix: await the result
const response = await fetchData();
console.log(response);
```

Common signs: works sometimes, fails under load, fails on slower machines, passes when you add a console.log (because logging adds a delay).

## Debugging Tools

Use the right tool for the situation:

**Print/log debugging**: Fast and effective for most problems. Add a `console.log` or `print()` at key points. Remove them when done.

```
> add some debug logging around the authentication flow so we can see
  what's happening
```

**Breakpoints and debuggers**: Better for complex state. Use when you need to inspect multiple variables at a specific point in execution.

**Network inspection**: For API issues, check the actual request and response. Use browser devtools, `curl`, or logging middleware.

**Git bisect**: When you know it used to work, `git bisect` finds the commit that broke it.

```
> help me use git bisect to find when the search feature broke — it was
  working as of last Tuesday
```

## When to Change Your Approach

If you have tried the same approach three times and it is not working, stop. Try a different angle:

- **Read the error message again**, literally. Often the answer is in the message but gets skimmed.
- **Check assumptions**: Is the function being called at all? Is the data what you think it is?
- **Simplify**: Strip the problem down to the smallest reproducing case.
- **Search**: Someone has likely hit this exact error before. Search the error message.
- **Step back**: Explain the problem out loud (to Claude, to a colleague, to a rubber duck). The act of explaining often reveals the issue.

## Log Reading

Claude can read and analyze logs effectively. Paste logs directly or point Claude at log files:

```
> read /var/log/app/error.log and tell me what's going wrong
```

```
> here are the last 50 lines of the application log — what pattern
  do you see in the errors?
```

Claude will identify patterns, correlate timestamps, and trace request flows through log entries. For large log files, let Claude read them with the Read tool rather than pasting them into the conversation — this uses context more efficiently.

## Fast Bug Triage Patterns

These patterns minimize context-switching between tools and let Claude handle the full triage loop.

### Slack-to-fix pipeline

With Slack MCP configured (see `core/mcp-guide.md`), paste a bug thread directly:

```
Pull the bug report from #bugs channel about the checkout failure. Read the thread, find the root cause, and fix it.
```

Zero context switching — Claude reads the report, explores the code, and implements the fix in one session.

### CI failure triage

Point Claude at the failure and get out of the way:

```
Go fix the failing CI tests. Don't ask me questions — read the test output, find the issue, fix it, and verify.
```

Don't micromanage. Claude reads the CI output, identifies the failing test, traces the root cause, and fixes it. Intervene only if it gets stuck after 2-3 attempts.

### Distributed system debugging

For multi-service issues, point Claude at container or service logs:

```
Read the docker logs for the api and worker containers. Correlate the timestamps to find why the job processing is failing.
```

```
Check the logs in /var/log/app/ across the api, queue, and worker services. What's the sequence of events leading to the timeout?
```

Claude can trace request flows across services when given access to the relevant log sources.
