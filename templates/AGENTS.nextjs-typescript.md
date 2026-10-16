# AGENTS.md

## Project
[PROJECT NAME]: [ONE LINE]. Next.js (App Router) + TypeScript. Package manager: [pnpm / npm / yarn]. Node [VERSION].

## Commands (run from the repo root)
- Install: `[pnpm install]`
- Dev server: `[pnpm dev]` (http://localhost:3000)
- Type-check: `[pnpm tsc --noEmit]`
- Lint: `[pnpm lint]`
- Tests: `[pnpm test]`; one file: `[pnpm test path/to/file.test.ts]`
- Production build: `[pnpm build]`
A task is not done until type-check, lint and the tests for the files you touched pass.

## Layout
- `app/` routes, layouts and route handlers. `components/` shared UI. `lib/` non-UI logic.
- Tests sit beside the file as `name.test.ts(x)`.

## Conventions
- Server Components by default. Add `"use client"` only for state, effects or browser APIs, and push it to the smallest leaf component.
- Fetch data in Server Components or route handlers, not in `useEffect`.
- TypeScript strict. No `any` (use `unknown` and narrow). No `!` non-null assertions without a comment saying why.
- Validate every external input (request body, search params, env vars) with [zod / YOUR VALIDATOR] at the boundary.
- Named exports for components. Next's special files (`page.tsx`, `layout.tsx`) keep their default exports.
- Styling: [Tailwind / CSS modules]. No inline styles.

## Boundaries
- Ask before adding a dependency or changing `[next.config.js]`, CI workflows or the database schema.
- Never edit `.next/`, lockfiles by hand, or generated files in `[generated/]`.
- Never read, print or commit `.env*`. Variable names live in `.env.example`.

## Git
- Small commits, imperative subject under 72 characters. Don't push or open PRs unless asked.
