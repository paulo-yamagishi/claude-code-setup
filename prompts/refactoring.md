# Refactoring Prompt Patterns

## Safe Refactoring

```
Refactor [target] to [goal]. Requirements:
- All existing tests must continue to pass
- No behavior changes — this is a pure refactoring
- Make changes incrementally, running tests after each step
- If a test fails, stop and assess before continuing
```

## Extract Pattern

```
Extract [logic] from [location] into a reusable [function/module/class].
- Identify all call sites
- Create the extraction with the same interface
- Update all call sites
- Verify with tests
```

## Rename/Move

```
Rename [old] to [new] across the entire codebase.
- Find all references (imports, usages, tests, docs)
- Update everything consistently
- Run tests to verify nothing broke
```

## Reduce Complexity

```
This [function/module] is too complex. Simplify it by:
- Breaking into smaller, focused functions
- Removing dead code paths
- Simplifying conditional logic
- Using early returns instead of deep nesting

Keep the same external behavior and ensure tests pass.
```

## Tips

- Always run tests before AND after refactoring
- Make one type of change at a time (rename, then restructure, then optimize)
- Use git commits between refactoring steps so you can revert if needed
- Let Claude explore the code first to understand the full impact
