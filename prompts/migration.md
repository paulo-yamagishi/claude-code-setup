# Migration Prompt Patterns

## Dependency Migration

```
Migrate from [old library] to [new library].

1. First, inventory all usages of [old library] in the codebase
2. Plan the migration order (start with least-dependent modules)
3. For each module:
   a. Update imports and API calls
   b. Run tests for that module
   c. Commit the change
4. Remove the old dependency
5. Run full test suite
```

## Framework Version Upgrade

```
Upgrade [framework] from v[old] to v[new].

1. Read the migration guide for v[new]
2. Identify breaking changes that affect our codebase
3. Plan changes needed, ordered by dependency
4. Make changes incrementally with tests between each step
5. Update configuration files (tsconfig, package.json, etc.)
```

## Language/Runtime Migration

```
Migrate [component] from [old language] to [new language].

1. Understand the existing behavior completely (read tests and code)
2. Set up the new project structure
3. Port module by module, writing tests first
4. Verify behavior parity with existing tests
5. Set up CI for the new codebase
```

## Database Migration

```
Create a database migration to [change].

1. Write the migration SQL/script
2. Write a rollback script
3. Test on a copy of the data
4. Verify the application works with the new schema
5. Document any required data backfill steps
```

## Tips

- Migrations are high-risk — commit frequently and test at every step
- Keep old and new running in parallel when possible
- Use feature flags for gradual rollouts
- Document what changed for other team members
