# Full-Stack Projects with Claude Code

## Common Stacks

| Stack | Frontend | Backend | DB |
|-------|----------|---------|-----|
| **Next.js** | React (built-in) | API Routes / Server Actions | Prisma / Drizzle |
| **React + Express** | React (Vite) | Express / Fastify | Prisma / Drizzle |
| **SvelteKit** | Svelte (built-in) | Built-in endpoints | Prisma / Drizzle |
| **Django + React** | React (Vite) | Django REST | Django ORM |

## Project Structure

### Monorepo (turborepo / nx)

```
myproject/
├── apps/
│   ├── web/            — frontend (Next.js, React, etc.)
│   └── api/            — backend (Express, Fastify, etc.)
├── packages/
│   ├── shared/         — shared types, utils
│   └── ui/             — shared UI components
├── turbo.json
├── package.json
└── CLAUDE.md
```

### Single-repo (Next.js / SvelteKit)

```
myproject/
├── src/
│   ├── app/            — pages and routes
│   ├── components/     — UI components
│   ├── lib/            — business logic, DB client
│   └── server/         — server-only code
├── prisma/
│   └── schema.prisma
└── CLAUDE.md
```

## API Design

### REST

- Use resource-based URLs: `/api/users/:id`
- Return consistent response shapes
- Use proper HTTP methods and status codes

### tRPC (TypeScript full-stack)

- End-to-end type safety without code generation
- Define routers on the server, call like functions on the client

## Database

- Use an ORM (Prisma, Drizzle) for type-safe queries
- Always use migrations, never modify the DB schema manually
- Seed data scripts for development

```bash
npx prisma migrate dev          # create and apply migration
npx prisma db seed              # seed development data
npx prisma studio               # visual DB browser
```

## Authentication

| Approach | When to use |
|----------|-------------|
| **Session-based** | Traditional web apps, server-rendered |
| **JWT** | API-first, mobile clients |
| **OAuth providers** | Social login (use next-auth, lucia, etc.) |

## Key Principles for Claude

- Keep frontend and backend concerns separated
- Share types between frontend and backend (never duplicate)
- Database access only happens server-side
- Validate all input at the API boundary (use zod)
- Environment variables for secrets (never hardcode)
