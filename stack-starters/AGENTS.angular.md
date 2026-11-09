# AGENTS.md — Angular (TypeScript)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** Node [20+ / 22 LTS], [npm / pnpm], Angular [19 / 20] with the Angular CLI, standalone components (no NgModules for new code), signals for local state, TypeScript in strict mode, tests with [Vitest / Karma + Jasmine / Jest], linting with [angular-eslint]. Replace bracketed defaults if your project differs, and check the current Angular docs for your version, since APIs change between majors.

## Project

- App: [PROJECT NAME] — [one-line purpose]
- Backend/API: [base URL or service] · auth: [provider / none]
- Rendering: [client-only SPA / SSR with @angular/ssr]
- Styling: [SCSS / Tailwind / Angular Material]

## Commands (repo root)

| Task | Command |
|------|---------|
| Install deps | `[npm ci]` |
| Dev server | `[npx ng serve]` → http://localhost:4200 |
| Lint | `[npx ng lint]` |
| Tests | `[npx ng test --watch=false]` |
| Production build | `[npx ng build]` (output in `dist/`) |
| Generate code | `[npx ng generate component path/name]` (match existing flags) |
| Deploy | `[your host command]` (ask first) |

A change is not done until lint, tests and `ng build` all pass with no new warnings.

## Layout

- `src/main.ts` — bootstraps the app · `src/app/app.config.ts` — providers (router, HttpClient, interceptors)
- `src/app/app.routes.ts` — route table · feature folders under `src/app/[feature]/`
- `src/app/core/` or `shared/` — services, interceptors, guards, shared UI (follow what exists)
- `src/environments/` — build-time config (public, not secret) · `public/` or `src/assets/` — static files
- `angular.json`, `tsconfig*.json`, `eslint.config.js` — tooling config
- Do not edit `dist/`, `.angular/`, `node_modules/`, or generated API clients by hand

## Conventions

1. **Follow the existing structure.** Match current folders, naming (`name.component.ts`, `name.service.ts` or the repo's newer style) and patterns. Don't add NgRx, a UI kit, or a new styling approach the repo doesn't already use.
2. **Standalone components.** New components, directives and pipes are standalone and import only what they use. Don't convert existing NgModules unless asked.
3. **Signals for state, RxJS for streams.** Use `signal`, `computed` and `input()`/`output()` for component state where the repo does; keep RxJS for HTTP and event streams. Don't mix both styles for the same piece of state.
4. **No leaked subscriptions.** Prefer the `async` pipe or `toSignal`; otherwise use `takeUntilDestroyed`. Never leave a manual `subscribe` without cleanup.
5. **Use the new control flow.** Write `@if`, `@for (item of items; track item.id)` and `@switch` in templates where the repo already does; always give `@for` a stable `track` expression.
6. **OnPush by default.** Use `ChangeDetectionStrategy.OnPush` for new components and avoid mutating inputs in place.
7. **HTTP through services.** Components call injected services; services use `HttpClient`. Handle loading and error states. Auth headers and error handling live in interceptors, not scattered calls.
8. **Typed forms.** Use typed reactive forms with validators; don't add `any` to form groups.
9. **TypeScript strict, no `any`.** Keep `strict` and `strictTemplates` on; don't silence errors with `@ts-ignore` or `$any()` in templates.
10. **Security.** Don't bypass sanitization with `bypassSecurityTrust*` on untrusted content, and never put secrets in `environment.ts` — everything there ships to the browser.
11. **Accessible by default.** Use semantic elements, label form controls, give images `alt` text, and keep everything keyboard reachable.
12. **Dependencies.** Ask before adding, removing, or upgrading a package, or running `ng update`.

## Testing

- Use `TestBed` with the component's real standalone imports; mock services with providers, and HTTP with `provideHttpClientTesting` and `HttpTestingController`.
- Test what a user sees (rendered text, roles, emitted outputs), not private fields.
- New behavior gets a test; a bug fix gets a test that fails before the fix where practical.

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by refactors or reformatting of untouched files
- [ ] Lint, tests and `ng build` pass with no new warnings
- [ ] No `any`, `@ts-ignore`, `$any()` or `bypassSecurityTrust*` added
- [ ] No manual subscriptions without cleanup; every `@for` has a stable `track`
- [ ] Loading, empty and error states handled for new HTTP calls
- [ ] No secrets in `environment*.ts`; new UI is keyboard accessible and labeled
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify)

## Boundaries — ask first

- Adding a dependency, running `ng update`, or changing Angular/TypeScript versions
- Changing `angular.json`, build budgets, SSR setup, or environment files
- Deploying or changing hosting settings

## How to customize

1. Fill `[BRACKETED]` values and delete options you don't use (SSR, Material, test runner).
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
