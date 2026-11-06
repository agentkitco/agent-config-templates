# AGENTS.md — Next.js (App Router) + TypeScript

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** Next.js App Router, TypeScript strict, React Server Components by default, Tailwind optional. Replace bracketed defaults if your project differs.

## Project

- App: [PROJECT NAME] — [one-line purpose]
- Runtime: Node [20+] · Package manager: [pnpm]
- Framework: Next.js App Router + TypeScript (strict)

## Commands (repo root)

| Task | Command |
|------|---------|
| Install | `pnpm install` |
| Dev | `pnpm dev` → http://localhost:3000 |
| Type-check | `pnpm tsc --noEmit` |
| Lint | `pnpm lint` |
| Unit / component tests | `pnpm test` · one file: `pnpm test path/to/file.test.ts` |
| Production build | `pnpm build` |

A change is not done until type-check, lint, and tests for touched files pass. Prefer the narrowest check while iterating; run the full set before handing off.

## Layout

- `app/` — routes, layouts, `loading.tsx` / `error.tsx`, route handlers under `app/api/`
- `components/` — shared UI (colocate route-only UI under `app/.../_components/`)
- `lib/` — non-UI helpers, data access, validators
- Tests beside source: `name.test.ts(x)` or under `__tests__/`

## Conventions

1. **Server Components by default.** Add `"use client"` only for state, effects, event handlers, or browser APIs — and push it to the smallest leaf.
2. **Data fetching** in Server Components or route handlers, not in `useEffect` for initial load.
3. **TypeScript strict.** No `any` (use `unknown` + narrow). Avoid `!` non-null assertions without a one-line why.
4. **Validate at boundaries.** Request bodies, search params, and env vars go through [zod] (or your validator) before use.
5. **Exports.** Named exports for components and helpers. Keep Next special-file default exports (`page.tsx`, `layout.tsx`, `route.ts`).
6. **Styling.** [Tailwind / CSS modules] only — no inline styles unless a third-party API forces it.
7. **Env.** Document names in `.env.example`. Never read, print, or commit `.env*` secrets.

## Review habits (before you say “done”)

- [ ] Diff is scoped to the ask — no drive-by refactors
- [ ] New/changed Server → Client boundaries are intentional
- [ ] Forms and API routes validate input; errors are typed and user-safe
- [ ] No secrets, tokens, or PII in logs or commits
- [ ] `tsc`, lint, and relevant tests pass
- [ ] If you opened a PR: blockers-first summary (what broke if wrong → what changed → how to verify)

## Boundaries — ask first

- Adding a dependency, changing `next.config.*`, CI, auth, or database schema
- Deleting public files, rewriting git history, force-pushing, or touching production infra

## Never

- Edit `.next/`, hand-edit lockfiles, or commit generated noise
- Disable lint/type rules project-wide to “make it green”
- Claim a task is done when checks failed

## How to customize

1. Fill `[BRACKETED]` values.
2. Delete sections that do not apply (keep the file short — agents follow short files better).
3. Optionally add a one-line `CLAUDE.md` with `@AGENTS.md` so Claude Code shares this file.

---

### Disclaimer

Written with AI assistance by [AgentKit Co](https://agentkitco.com/). Starting point, not a guarantee — review against your repo and keep real enforcement in CI.

### Want more?

- Free templates + generator: https://agentkitco.com/tools/agents-md-generator/
- [AI Coding Agent Config Kit](https://agentkitco.com/products/agent-config-kit/) ($14) — 12-stack AGENTS.md set, Cursor rules, Copilot files
- [AI PR Review Kit](https://agentkitco.com/products/ai-pr-review-kit/) ($17) — blockers-first checklists, paste-ready PR comments, review prompts for Claude Code & Cursor

License: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — use and adapt with attribution to AgentKit Co.
