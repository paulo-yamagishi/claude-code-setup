# Testing Rules

## Requirements
- All new features must have tests
- Bug fixes must include a regression test
- Run the full test suite before committing

## Conventions
- Test file naming: `*.test.ts` or `test_*.py`
- One test file per module
- Use descriptive test names that explain the scenario
- Arrange-Act-Assert pattern

## What to Test
- Public API surface
- Edge cases and error conditions
- Integration points
- User-facing behavior (not implementation details)

## What NOT to Test
- Private implementation details
- Third-party library internals
- Trivial getters/setters
