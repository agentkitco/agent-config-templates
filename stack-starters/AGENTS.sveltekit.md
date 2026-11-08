# AGENTS.md — SvelteKit (Svelte 5)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** SvelteKit 2, Svelte [5] with runes, TypeScript strict, Vite, package manager [pnpm], adapter [auto / node / cloudflare / vercel]. Replace bracketed defaults if your project differs. If components still use Svelte 4 syntax (`export let`, `$:`, `on:click`), match the file you are editing — don't migrate a file to runes unless asked.

## Project

- App: [PROJECT NAME] — [one-line purpose]
- Runtime: Node [20+] · SvelteKit 2 · Svelte [5] · TypeScript (strict)
- Package manager: [pnpm]
- Adapter: [adapter-auto] · styling: [Tailwind / CSS]

## Commands (repo root)

| Task | Command |
|------|---------|
| Install | `pnpm install` |
| Dev | `pnpm dev` → http://localhost:5173 |
| Type-check | `pnpm check` (`svelte-kit sync && svelte-check`) |
| Lint / format | `pnpm lint` · `pnpm format` |
| Unit / component tests | `pnpm test:unit` · one file: `pnpm vitest run path/to/file.test.ts` |
| E2E tests | `pnpm test:e2e` (Playwright) |
| Production build | `pnpm build` · preview: `pnpm preview` |

A change is not done until `pnpm check`, lint, and tests for touched files pass. Run `pnpm build` before handing off a routing, load-function, hook, or adapter change.

## Layout

- `src/routes/` — file-based routes
  - `+page.svelte` / `+layout.svelte` — UI
  - `+page.ts` / `+layout.ts` — universal `load` (runs on server and in the browser)
  - `+page.server.ts` / `+layout.server.ts` — server-only `load` and form `actions`
  - `+server.ts` — API endpoints (`GET`, `POST`, ... returning `Response` / `json()`)
  - `(group)/` folders group routes without changing the URL; `[param]`, `[[optional]]`, `[...rest]` for params
- `src/lib/` — shared code, imported as `$lib/...`
- `src/lib/server/` — server-only code (DB, secrets); SvelteKit blocks importing it from browser code
- `src/hooks.server.ts` — `handle`, `handleFetch`, `handleError` (auth, locals, logging)
- `src/app.d.ts` — `App.Locals`, `App.PageData`, `App.Error` types
- `src/app.html` — HTML shell
- `static/` — files served as-is
- `svelte.config.js` / `vite.config.ts` — adapter, aliases, preprocessors
- Do not edit `.svelte-kit/` or `build/` (generated)

## Conventions

1. **Follow the existing structure.** Match current route grouping, naming, and patterns. Don't introduce a new state library, UI kit, ORM, or i18n setup unless the repo already uses it.
2. **Runes in Svelte 5 files.** Use `$state`, `$derived`, `$effect`, `$props`, and `onclick={...}` event attributes. Prefer `$derived` over `$effect` for computed values; `$effect` is for side effects only. Shared reactive state in `.svelte.ts` modules.
3. **Load data in `load`, not `onMount`.** First-paint data comes from `+page.ts` / `+page.server.ts` so it renders during SSR. Use the `fetch` passed into `load` (not global `fetch`) so cookies and relative URLs work and responses are inlined.
4. **Server-only when it touches secrets.** Database calls, private API keys, and anything with credentials go in `+page.server.ts`, `+server.ts`, hooks, or `$lib/server/`. Never in universal `load` or components.
5. **Forms use actions.** Mutations go through form `actions` in `+page.server.ts` with `use:enhance` for progressive enhancement. Return `fail(400, { ... })` for validation errors; use `redirect()` and `error()` from `@sveltejs/kit` (in SvelteKit 2 they are thrown for you — don't wrap them in `throw`, and don't swallow them in a `try/catch`).
6. **No shared server state.** Don't keep per-user data in module-level variables or stores on the server — they leak across requests. Per-request data goes in `event.locals` (typed in `app.d.ts`) or is returned from `load`.
7. **Browser-only APIs.** Touch `window`, `document`, `localStorage` only in `onMount`, `$effect`, or behind `browser` from `$app/environment`.
8. **TypeScript strict.** Use the generated `./$types` (`PageLoad`, `PageServerLoad`, `Actions`, `RequestHandler`). No `any` (use `unknown` + narrow); avoid `!` assertions without a one-line why.
9. **Validate at boundaries.** Form data, JSON bodies, and URL params go through [zod / valibot] (or the repo's validator) before use.
10. **Env vars.** Secrets via `$env/static/private` or `$env/dynamic/private` (server only). Browser-safe values must be prefixed `PUBLIC_` and read from `$env/static/public`. Document names in `.env.example`. Never read, print, or commit `.env*` secrets.
11. **Navigation and SEO.** Internal links are plain `<a href>` (SvelteKit handles client routing); use `goto` from `$app/navigation` only for programmatic navigation. Titles and meta go in `<svelte:head>`. Page options (`prerender`, `ssr`, `csr`, `trailingSlash`) are exported from `+page.ts` / `+layout.ts`.
12. **Dependencies.** Ask before adding an npm package, Vite plugin, or Svelte preprocessor. Prefer libraries already in the tree.

## Testing

- Vitest for `$lib` utils, validators, and `.svelte.ts` state; [@testing-library/svelte / vitest-browser-svelte] for components if the repo already does.
- Playwright for routes, form actions, auth, and anything that depends on SSR + hydration.
- New behavior gets a test; a bug fix gets a test that fails before the fix.

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by refactors, rune migrations, or reformatting of untouched files
- [ ] First-paint data comes from `load`, using the provided `fetch`
- [ ] Secrets and DB access only in server files or `$lib/server/`; nothing private in `PUBLIC_` env vars
- [ ] No module-level per-user server state; browser APIs guarded
- [ ] Form actions validate input and return `fail()` with field errors; `redirect()` / `error()` not swallowed
- [ ] `$types` used for load/action/handler signatures
- [ ] `pnpm check`, lint, relevant tests, and `pnpm build` (for routing/hook/adapter changes) pass
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify)

## Boundaries — ask first

- Adding a dependency, changing `svelte.config.js`, the adapter, `vite.config.ts`, CI, auth hooks, or the database schema
- Migrating Svelte 4 components to runes, or changing `ssr` / `prerender` / `csr` defaults for a layout
- Deleting public files, rewriting git history, force-pushing, or touching production infra or deploy config

## Never

- Edit `.svelte-kit/` or `build/`, hand-edit lockfiles, or commit generated noise
- Import `$lib/server/` or `$env/*/private` from browser code, or work around SvelteKit's server-only import guard
- Disable lint/type rules project-wide to "make it green"
- Log tokens, cookies, or PII, or claim a task is done when `pnpm check`, tests, or the build failed

## How to customize

1. Fill `[BRACKETED]` values and delete options you don't use.
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
