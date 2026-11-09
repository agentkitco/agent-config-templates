# AGENTS.md — Vite + React (TypeScript)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** Node [20+ / 22 LTS], [npm / pnpm], Vite [6 / 7], React [18 / 19], TypeScript in strict mode, routing via [React Router / TanStack Router / none], data fetching via [TanStack Query / fetch], tests with [Vitest + React Testing Library], linting with [ESLint]. This is a client-side single-page app. Replace bracketed defaults if your project differs, and check the current Vite and React docs for your version.

## Project

- App: [PROJECT NAME] — [one-line purpose]
- Backend/API: [base URL or service] · auth: [provider / none]
- Styling: [Tailwind / CSS Modules / other]
- Hosting: [static host, e.g. Netlify / Vercel / Cloudflare Pages / S3]

## Commands (repo root)

| Task | Command |
|------|---------|
| Install deps | `[npm ci]` |
| Dev server | `[npm run dev]` → http://localhost:5173 |
| Type check | `[npx tsc -b --noEmit]` (or the repo's `typecheck` script) |
| Lint | `[npm run lint]` |
| Tests | `[npx vitest run]` (one file: `npx vitest run src/path.test.tsx`) |
| Production build | `[npm run build]` (output in `dist/`) |
| Preview build | `[npm run preview]` |
| Deploy | `[your host command]` (ask first) |

A change is not done until type check, lint, tests and `npm run build` all pass.

## Layout

- `index.html` — Vite entry; `src/main.tsx` mounts the app
- `src/components/` — shared UI components · `src/pages/` or `src/routes/` — route-level screens
- `src/hooks/` — custom hooks · `src/lib/` or `src/api/` — API client, utilities
- `public/` — static files copied as-is · `src/assets/` — imported assets (hashed by Vite)
- `vite.config.ts`, `tsconfig*.json`, `eslint.config.js` — tooling config
- Do not edit `dist/`, `node_modules/`, or generated API clients by hand

## Conventions

1. **Follow the existing structure.** Match current folders, naming, and patterns. Don't add a state library, router, UI kit, or CSS approach the repo doesn't already use.
2. **TypeScript strict, no `any`.** Type props and API responses. Prefer `unknown` plus narrowing over `any`; don't silence errors with `@ts-ignore`.
3. **Function components and hooks.** Follow the Rules of Hooks; keep effect dependency arrays complete rather than disabling the lint rule.
4. **Don't use effects for derived data.** Compute values during render (or `useMemo` if measured slow). Use effects only to sync with something outside React.
5. **Server data through one path.** Fetch through the repo's API client or query library, handle loading and error states, and cancel or ignore stale requests. Don't scatter raw `fetch` calls in components.
6. **Env vars are public.** Only `VITE_`-prefixed variables reach the client via `import.meta.env`, and everything in the bundle is readable by users. Never put secrets or private API keys in client code or `.env` files that ship.
7. **Escape user content.** Never use `dangerouslySetInnerHTML` on untrusted content without a sanitizer the repo already uses.
8. **Accessible by default.** Use semantic elements (`button`, `label`, `nav`), give images `alt` text, and keep everything keyboard reachable.
9. **Keys and lists.** Use stable ids as `key`, not array indexes, for lists that can change.
10. **Watch the bundle.** Lazy-load heavy routes with `React.lazy` / dynamic `import()`; don't pull in a large dependency for one small function.
11. **Dependencies.** Ask before adding, removing, or upgrading a package, or changing Vite, React, or TypeScript versions.

## Testing

- Use Vitest with React Testing Library; query by role or label (`getByRole`, `getByLabelText`) and drive interactions with `@testing-library/user-event`.
- Test behavior a user sees, not implementation details or internal state.
- Mock the network at the boundary ([MSW] or the repo's existing approach), not individual hooks.
- New behavior gets a test; a bug fix gets a test that fails before the fix where practical.

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by refactors or reformatting of untouched files
- [ ] Type check, lint, tests and `npm run build` pass
- [ ] No `any`, `@ts-ignore`, or disabled hook-dependency warnings added
- [ ] Loading, empty and error states handled for new data fetching
- [ ] No secrets in client code or `VITE_` variables; no unsanitized `dangerouslySetInnerHTML`
- [ ] New UI is keyboard accessible and labeled
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify)

## Boundaries — ask first

- Adding a dependency or upgrading Vite, React, TypeScript, or the router
- Changing build config, env handling, or the API base URL
- Deploying or changing hosting settings

## How to customize

1. Fill `[BRACKETED]` values and delete options you don't use (router, query library, MSW).
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
