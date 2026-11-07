# AGENTS.md — NestJS (Node.js + TypeScript)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** NestJS [10+], Node [20+ LTS], TypeScript, package manager [npm / pnpm / yarn]. Replace bracketed defaults if your project differs.

## Project

- App: [PROJECT NAME] — [one-line purpose]
- Runtime: Node [20+ LTS] · NestJS [10+] · TypeScript
- Package manager: [npm / pnpm / yarn]
- Shape: [HTTP API / microservice / hybrid]

## Commands (repo root)

| Task | Command |
|------|---------|
| Install | `[npm install / pnpm install / yarn]` |
| Dev server | `npm run start:dev` (or `pnpm` / `yarn` equivalent) |
| Build | `npm run build` |
| Prod start | `npm run start:prod` |
| Unit tests | `npm run test` · one file: `npm run test -- [path]` |
| E2E tests | `npm run test:e2e` |
| Lint | `npm run lint` |
| Format | `npm run format` |

A change is not done until unit tests (and e2e when you touched HTTP contracts) plus lint for touched modules pass. Prefer the narrowest test path while iterating; run the full suite before handing off.

## Layout

- `src/main.ts` — bootstrap (thin; wire global pipes / filters here)
- `src/app.module.ts` — root module
- `src/[feature]/` — feature modules (`*.module.ts`, `*.controller.ts`, `*.service.ts`)
- `src/[feature]/dto/` — request/response DTOs (`class-validator` / `class-transformer`)
- Pipes, guards, interceptors, filters — prefer feature-local; promote to `src/common/` only when shared
- `test/` or `e2e/` — end-to-end tests (black-box against the HTTP API)

## Conventions

1. **Modules own features.** One Nest module per feature domain. Export only what other modules must inject; keep controllers out of shared exports.
2. **DI via constructor.** Inject providers through the constructor; prefer interfaces / abstract tokens for swappable deps. Avoid `ModuleRef` lookups unless dynamic.
3. **Validation.** Use a global `ValidationPipe` (`whitelist`, `forbidNonWhitelisted`, `transform`). Validate at the edge with typed DTOs — do not re-parse raw bodies in services.
4. **DTOs.** Prefer `class-validator` + `class-transformer` classes over loose interfaces for inbound HTTP. Keep DTOs thin; map to domain types in the service layer.
5. **Config.** Load via `@nestjs/config` `ConfigModule` once at startup; fail fast on missing required keys. Never log secrets, tokens, or PII.
6. **HTTP.** Controllers stay thin (parse → call service → map response). Map domain errors to stable status codes via filters. Prefer Nest CLI generators (`nest g module|controller|service`) over hand-rolled boilerplate.
7. **Dependencies.** Ask before adding a package. Prefer libraries already in the tree. Pin via the lockfile only through the project’s package manager.

## Review habits (before you say “done”)

- [ ] Diff is scoped to the ask — no drive-by refactors
- [ ] New routes have DTO validation and a clear auth/guard story when the module already uses one
- [ ] No secrets, tokens, or PII in logs or commits
- [ ] `lint`, unit tests, and e2e (when contracts changed) pass on touched modules
- [ ] If you opened a PR: blockers-first summary (what broke if wrong → what changed → how to verify)

## Boundaries — ask first

- Adding a dependency, changing Node / Nest major version, CI, auth, or datastore schema
- Deleting public modules, rewriting git history, force-pushing, or touching production infra

## Never

- Commit `node_modules/`, `.env` with real secrets, or edit the lockfile by hand
- Disable eslint / prettier rules broadly to silence warnings (`eslint-disable` only with a one-line why)
- Claim a task is done when tests or lint failed
- Bypass the ValidationPipe or accept untyped `any` bodies at the HTTP edge

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
