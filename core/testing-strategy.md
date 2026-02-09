# Verification-First Testing Strategy

## Core Principle: Test Before You Touch

Always run existing tests before making any changes. This establishes a baseline so you know whether failures are yours or pre-existing.

```bash
# First thing in any session
claude "run the test suite and tell me the current state"
```

## Put Test Commands in CLAUDE.md

Claude needs to know how to run your tests. Add commands to your `CLAUDE.md`:

```markdown
# Testing
- Run all tests: `pytest`
- Run single test file: `pytest tests/test_auth.py`
- Run with coverage: `pytest --cov=src`
- Run fast tests only: `pytest -m "not slow"`
```

This avoids Claude guessing or asking you how to run tests every session.

## Testing Workflow with Claude Code

The recommended pattern follows a strict order:

### 1. Run Tests First

Understand the current state before doing anything.

```
> run the tests and summarize the results
```

Claude will execute the test suite, parse the output, and tell you what passes and what fails.

### 2. Write a Failing Test

For bugs, write a test that reproduces the problem. For features, write a test that describes the desired behavior.

```
> write a test that reproduces the bug in issue #42 — the user signup
  should reject emails without a TLD
```

### 3. Implement the Fix or Feature

```
> now fix the email validation to make that test pass
```

### 4. Run Tests Again

```
> run the full test suite to make sure nothing is broken
```

Claude verifies both that the new test passes and that no existing tests regressed.

## Test-Driven Workflow

Claude can write and run tests in the same session. This is one of its strongest capabilities. A single prompt can drive the full TDD cycle:

```
> write a failing test for the discount calculation when cart total exceeds
  $100, then implement the logic to make it pass, then run all tests
```

Claude will:
1. Read existing test files to match conventions
2. Write the failing test
3. Run it to confirm it fails
4. Implement the code
5. Run all tests to confirm everything passes

## Common Test Frameworks and Commands

| Language   | Framework | Run All          | Run One File                  |
|------------|-----------|------------------|-------------------------------|
| Python     | pytest    | `pytest`         | `pytest tests/test_foo.py`    |
| JavaScript | Jest      | `npx jest`       | `npx jest foo.test.js`        |
| TypeScript | Vitest    | `npx vitest run` | `npx vitest run foo.test.ts`  |
| Go         | go test   | `go test ./...`  | `go test ./pkg/foo/`          |
| Rust       | cargo     | `cargo test`     | `cargo test test_name`        |
| Ruby       | RSpec     | `bundle exec rspec` | `bundle exec rspec spec/foo_spec.rb` |

## Handling Flaky Tests

Flaky tests erode confidence. Handle them deliberately:

- **Identify**: A test that passes sometimes and fails sometimes is flaky. Run it 5 times to confirm.
- **Isolate**: Run the test alone vs. with the full suite. Flakiness often comes from shared state or ordering.
- **Fix or skip with a ticket**: Either fix the root cause or skip with `@pytest.mark.skip(reason="flaky — see #123")`. Never silently ignore.
- **Common causes**: Shared mutable state, time-dependent logic, network calls, race conditions, filesystem ordering.

```
> this test fails intermittently — help me identify the root cause
```

## Integration Tests vs Unit Tests

**Unit tests**: Fast, isolated, test a single function or class. Mock external dependencies. Run these constantly.

**Integration tests**: Slower, test components working together. Use real databases, APIs, or filesystems. Run these before merging.

```
# Run unit tests during development (fast feedback)
pytest tests/unit/

# Run integration tests before merging (thorough verification)
pytest tests/integration/
```

Use unit tests for logic. Use integration tests for wiring. Both are necessary.

## Anti-Patterns

**Skipping tests to make CI green**: This hides problems. Fix the test or fix the code.

**Testing implementation details**: Don't test that a private method was called. Test that the public behavior is correct. Implementation tests break on every refactor.

**Ignoring test failures**: A red test suite that everyone ignores is worse than no tests at all. If a test fails, either fix it or delete it.

**Writing tests after the fact just for coverage**: Tests written to hit a coverage number without thought tend to test nothing meaningful. Write tests that would catch real bugs.

**Not running tests locally before pushing**: CI is not your first line of defense. Run tests locally. Put the command in CLAUDE.md so Claude does this automatically.
