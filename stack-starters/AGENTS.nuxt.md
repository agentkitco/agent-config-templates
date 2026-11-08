# AGENTS.md — Nuxt (Vue 3)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** Nuxt [4 / 3], Vue 3 `<script setup lang="ts">`, TypeScript strict, Nitro server routes, package manager [pnpm]. Replace bracketed defaults if your project differs. If the repo is still on the Nuxt 3 root layout (`pages/`, `components/`, `composables/` next to `nuxt.config.ts`), follow that layout — don't move files into `app/` unless asked.

## Project

- App: [PROJECT NAME] — [one-line purpose]
- Runtime: Node [20+] · Nuxt [4 / 3] · Vue 3 · TypeScript (strict)
- Package manager: [pnpm]
- Rendering: [SSR / hybrid] · styling: [Tailwind / CSS]

## Commands (repo root)

| Task | Command |
|------|---------|
| Install | `pnpm install` |
| Dev | `pnpm dev` → http://localhost:3000 |
| Type-check | `pnpm nuxi typecheck` |
| Lint | `pnpm lint` |
| Unit / component tests | `pnpm test` · one file: `pnpm test path/to/file.test.ts` |
| Production build | `pnpm build` |
| Preview production build | `pnpm preview` |

A change is not done until type-check, lint, and tests for touched files pass. Prefer the narrowest check while iterating; run the full set (including `pnpm build` before handing off a routing or server-route change).

## Layout

Nuxt 4 default (adjust if this repo uses the Nuxt 3 root layout):

- `app/pages/` — file-based routes (`index.vue`, `[id].vue`, catch-alls)
- `app/components/` — Vue components (auto-imported)
- `app/composables/` — Vue composables (top-level files auto-imported; nested folders may need an explicit import)
- `app/layouts/` — page wrappers; set with `definePageMeta({ layout: '...' })`
- `app/middleware/` — route middleware (client/SSR navigation)
- `app/plugins/` — Vue/Nuxt plugins
- `app/utils/` — shared app helpers
- `app/app.vue` — root component (keep `<NuxtLayout>` / `<NuxtPage>` as peers do)
- `server/api/` — Nitro handlers at `/api/*` (`defineEventHandler`)
- `server/routes/` — Nitro handlers without the `/api` prefix
- `server/middleware/` — runs on every server request; inspect or extend, don't send the response
- `shared/` — isomorphic code safe for both the Vue app and the Nitro server
- `public/` — static files served as-is
- `nuxt.config.ts` — modules, `runtimeConfig`, build settings
- `test/` — unit, Nuxt runtime, and e2e tests
- Do not edit `.nuxt/` or `.output/` (generated)

## Conventions

1. **Follow the existing structure.** Match the current `app/` vs root `pages/` layout, naming, and patterns. Don't introduce Pinia, `@nuxt/content`, i18n, or a new UI kit unless the codebase already uses them.
2. **`<script setup lang="ts">`.** Composition API only, unless a file already uses Options API. Named exports for helpers; pages and components stay SFCs.
3. **Auto-imports.** Vue APIs (`ref`, `computed`, `watch`), Nuxt composables (`useFetch`, `useRoute`, `useRuntimeConfig`, `useSeoMeta`), and top-level `components/` / `composables/` are auto-imported. Don't add redundant imports of those. Nested composable folders are not auto-imported — import them explicitly.
4. **Data fetching.** Load initial data with `useFetch` or `useAsyncData` so it runs during SSR. Don't fetch the first page of data in `onMounted` + `$fetch` (that skips SSR and causes waterfalls). Use `$fetch` for client-only actions (form submits, button clicks).
5. **SSR-safe state.** Shared client+server state goes in `useState` (or the repo's existing store). Don't create module-level `ref`s — they leak across requests. Touch `window`, `document`, or other browser APIs only inside `onMounted` or behind `import.meta.client` (not the deprecated `process.client`).
6. **TypeScript strict.** No `any` (use `unknown` + narrow). Avoid `!` non-null assertions without a one-line why.
7. **Validate at boundaries.** Request bodies, query params, and form input go through [zod] (or your validator) in `server/api` (`readValidatedBody` / `getValidatedQuery` when the repo already uses them) before use.
8. **Server vs app.** `server/` is Nitro, not Vue. Don't import Vue components, `useFetch`, or other app composables into `server/`. Put code both sides need in `shared/`.
9. **Config and secrets.** Public values go in `runtimeConfig.public` and are safe to expose to the browser. Secrets stay in private `runtimeConfig` (or the environment) and are read only on the server. Document names in `.env.example`. Never read, print, or commit `.env*` secrets.
10. **Navigation and SEO.** Internal links use `<NuxtLink>`. Set titles and social tags with `useSeoMeta` / `useHead`. Route meta (layout, middleware, `ssr`) goes in `definePageMeta`.
11. **Styling.** [Tailwind / CSS] only — no inline styles unless a third-party API forces it.
12. **Dependencies.** Ask before adding an npm package or a Nuxt module (`nuxt.config` `modules` array). Prefer libraries already in the tree.

## Testing

- Unit tests for composables, utils, and validators; [@nuxt/test-utils + Vitest / Playwright] as the repo already does.
- New behavior gets a test; a bug fix gets a test that fails before the fix.
- A hydration mismatch or a `server/api` contract change needs a runtime or e2e check, not only a unit test.

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by refactors or reformatting of untouched files
- [ ] Initial data uses `useFetch` / `useAsyncData`; no `onMounted` fetch for first paint
- [ ] No browser-only APIs during setup; no module-level `ref` state
- [ ] Server handlers validate input and don't import Vue/app composables
- [ ] Secrets are not in `runtimeConfig.public`, logs, fixtures, or commits
- [ ] Internal links use `<NuxtLink>`; SEO tags updated when the page is user-facing
- [ ] `nuxi typecheck`, lint, relevant tests, and `pnpm build` (for routing or server-route changes) pass
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify)

## Boundaries — ask first

- Adding a dependency or Nuxt module, or changing `nuxt.config.ts`, CI, auth, or the database schema
- Switching Nuxt 3 root layout ↔ Nuxt 4 `app/` directory, or changing `ssr` / rendering mode
- Deleting public files, rewriting git history, force-pushing, or touching production infra (Nitro preset, Cloudflare/Vercel/Node deploy config)

## Never

- Edit `.nuxt/` or `.output/`, hand-edit lockfiles, or commit generated noise
- Disable lint/type rules project-wide to "make it green"
- Expose secrets via `runtimeConfig.public` or log tokens, cookies, or PII
- Claim a task is done when type-check, tests, or the production build failed

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
