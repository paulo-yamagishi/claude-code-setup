# TypeScript Projects with Claude Code

## Tooling

| Tool | Purpose |
|------|---------|
| **npm/pnpm/bun** | Package manager |
| **tsc** | Type checker (via `tsconfig.json`) |
| **eslint** | Linter |
| **prettier** | Formatter |
| **vitest** / **jest** | Test runner |

## Project Structure

```
myproject/
├── src/
│   ├── index.ts
│   ├── types/
│   │   └── index.ts
│   └── utils/
│       └── helpers.ts
├── tests/
│   └── helpers.test.ts
├── tsconfig.json
├── package.json
└── CLAUDE.md
```

## TypeScript Configuration

Use strict mode. Key `tsconfig.json` settings:

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "target": "ES2022",
    "module": "NodeNext",
    "outDir": "dist"
  }
}
```

## Style Conventions

- **strict mode** always enabled
- Explicit return types on all exported functions
- Prefer `interface` over `type` for object shapes
- Use `unknown` instead of `any`
- Prefer `const` assertions and `as const`
- Use barrel exports (`index.ts`) sparingly

```typescript
export interface User {
  id: string;
  name: string;
  role: "admin" | "user";
}

export function getUser(id: string): User | undefined {
  // ...
}
```

## Testing

```typescript
// vitest example
import { describe, it, expect } from "vitest";
import { getUser } from "../src/users";

describe("getUser", () => {
  it("returns undefined for unknown id", () => {
    expect(getUser("unknown")).toBeUndefined();
  });
});
```

## Common Commands

```bash
npx tsc --noEmit               # type check only
npx vitest                      # run tests (watch mode)
npx vitest run                  # run tests once
npx eslint .                    # lint
npx prettier --write .          # format
```

## Build Tools

- **tsc** — standard compiler, good for libraries
- **esbuild** — fast bundler for applications
- **tsup** — zero-config library bundler (wraps esbuild)
- **vite** — dev server + bundler for web apps

## Common Patterns

- **zod** for runtime validation and type inference
- **Discriminated unions** for state modeling
- **Generics** for reusable utilities
- **Template literal types** for string patterns
- **satisfies** operator for type-safe config objects

```typescript
// Discriminated union
type Result<T> =
  | { ok: true; value: T }
  | { ok: false; error: string };

// Zod schema with inferred type
const UserSchema = z.object({ name: z.string(), age: z.number() });
type User = z.infer<typeof UserSchema>;
```
