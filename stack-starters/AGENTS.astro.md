# AGENTS.md — Astro (TypeScript)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** Astro [5.x], TypeScript (`astro/tsconfigs/strict`), package manager [pnpm], content in [content collections via `src/content.config.ts`], UI islands in [React / Svelte / Vue / Solid / none], output [static / server with `@astrojs/[node|cloudflare|vercel|netlify]` adapter]. Replace bracketed defaults if your project differs. Check the current Astro docs for your version, since APIs move between majors.

## Project

- Site: [PROJECT NAME] — [one-line purpose]
- Astro [5.x] · TypeScript strict · output: [static]
- Package manager: [pnpm]
- Styling: [Tailwind / scoped `<style>` / CSS modules] · integrations: [`@astrojs/mdx`, `@astrojs/sitemap`, ...]

## Commands (repo root)

| Task | Command |
|------|---------|
| Install | `pnpm install` |
| Dev | `pnpm dev` (`astro dev` → http://localhost:4321) |
| Type-check | `pnpm astro check` |
| Regenerate types | `pnpm astro sync` (after changing collections, env schema, or actions) |
| Build | `pnpm build` (`astro build` → `dist/`) |
| Preview build | `pnpm preview` |
| Lint / format | `[pnpm lint]` · `[pnpm format]` |
| Tests | `[pnpm vitest run]` · e2e: `[pnpm playwright test]` |
| Deploy | `[your host's command]` (ask first) |

A change is not done until `astro check` and `astro build` pass. A broken build is the most common failure on Astro sites, because content schema errors and bad imports only show up there.

## Layout

- `src/pages/` — file-based routes (`.astro`, `.md`, `.mdx`); `src/pages/api/*.ts` for endpoints (`GET`, `POST` exports)
- `src/layouts/` — page shells (head, nav, footer); one `BaseLayout.astro` sets `<title>`, meta, canonical
- `src/components/` — `.astro` components (zero JS by default) and framework islands
- `src/content/` (or the `base` in loaders) — Markdown/MDX/JSON entries; schemas in `src/content.config.ts`
- `src/actions/` — Astro Actions if the repo uses them
- `src/middleware.ts` — request middleware (server output only)
- `public/` — copied as-is (favicons, `robots.txt`); images that need optimizing go in `src/assets/`
- `astro.config.mjs` — integrations, adapter, `site`, redirects
- Do not edit `dist/`, `.astro/`, `node_modules/`, or generated `env.d.ts` types

## Conventions

1. **Follow the existing structure.** Match current folders, layouts, and integration choices. Don't add a UI framework, CSS system, or CMS the repo doesn't already use.
2. **Ship zero JS by default.** Write `.astro` components first. Only reach for a framework island when the UI is truly interactive, and pick the lightest directive: `client:visible` or `client:idle` before `client:load`. Never add `client:*` just to make something render.
3. **Know where code runs.** The component frontmatter (`---` fence) runs at build time (static) or on the server (on-demand). Browser APIs (`window`, `localStorage`) only work inside islands or `<script>` tags. Don't import server secrets into islands.
4. **Content collections are typed.** Add fields in `src/content.config.ts` with zod and run `astro sync`. Use `getCollection` / `getEntry` and `render(entry)`; don't `import.meta.glob` Markdown when a collection exists. Filter drafts (`data.draft`) in production listings.
5. **Static routes need `getStaticPaths`.** Dynamic routes like `[slug].astro` in static output must return every path. Keep slugs stable; renaming one changes a public URL, so add a redirect in `astro.config.mjs`.
6. **Images go through `astro:assets`.** Use `<Image />` / `<Picture />` with files in `src/assets/` and always set `alt` (empty `alt=""` only for decorative images). Don't hot-link large images from `public/`.
7. **Environment variables.** Use the `astro:env` schema if the repo has one; otherwise only `PUBLIC_*` values are allowed on the client. Never put secrets in a `PUBLIC_` variable or commit a real `.env`.
8. **On-demand routes are explicit.** In static output, opt a page into server rendering with `export const prerender = false` only when it needs request data, and only if an adapter is installed. Ask before switching the whole site's `output`.
9. **SEO basics stay intact.** Every page sets a unique `<title>` and description through the layout. Keep `site` in `astro.config.mjs` correct so canonical URLs and `@astrojs/sitemap` are right.
10. **Scripts and styles.** Prefer scoped `<style>` and processed `<script>` tags. Use `is:inline` only when a script must run unbundled (for example a theme flash fix) and say why.
11. **Endpoints validate input.** API routes and Actions validate bodies (Actions `input` with zod, or the repo's validator) and return proper status codes. No stack traces in responses.
12. **Dependencies.** Ask before adding an npm package or an Astro integration (`astro add` edits config files too).

## Testing

- `astro check` and `astro build` are the baseline checks for every change.
- Unit-test helpers and islands with [Vitest]; Astro components can be rendered with the Container API (`experimental_AstroContainer`) if the repo already does so.
- For pages and links, run [Playwright] against `astro preview` when the change touches navigation or forms.
- New behavior gets a test; a bug fix gets a test that fails before the fix where practical.

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by refactors or reformatting of untouched files
- [ ] `astro check` and `astro build` pass; no new content schema errors
- [ ] No unnecessary `client:*` islands; interactive bits use the lightest directive
- [ ] No browser APIs in frontmatter; no secrets in `PUBLIC_` vars or islands
- [ ] Images use `astro:assets` with real `alt` text
- [ ] Changed or removed URLs have redirects; titles, descriptions, and canonical tags still render
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify)

## Boundaries — ask first

- Adding a dependency or integration, changing the adapter or `output` mode, or editing `astro.config.mjs` redirects
- Renaming or deleting published pages or content slugs
- Deploying, changing hosting settings, or changing production environment variables

## How to customize

1. Fill `[BRACKETED]` values and delete options you don't use (static vs server, island framework).
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
