# AGENTS.md — Express (Node.js + TypeScript)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** Express [4 / 5], Node [20+], TypeScript strict, package manager [pnpm], validation with [zod], data layer [Prisma / Drizzle / Knex / none], tests with [Vitest / Jest] + supertest. Replace bracketed defaults if your project differs. If the repo is plain JavaScript, match it — don't convert files to TypeScript unless asked.

## Project

- Service: [PROJECT NAME] — [one-line purpose]
- Runtime: Node [20+] · Express [5] · TypeScript (strict)
- Package manager: [pnpm]
- Database: [Postgres via Prisma] · auth: [session / JWT / none]

## Commands (repo root)

| Task | Command |
|------|---------|
| Install | `pnpm install` |
| Dev (watch) | `pnpm dev` → http://localhost:[3000] |
| Type-check | `pnpm typecheck` (`tsc --noEmit`) |
| Lint / format | `pnpm lint` · `pnpm format` |
| Tests | `pnpm test` · one file: `pnpm vitest run path/to/file.test.ts` |
| Build | `pnpm build` → `dist/` · run: `pnpm start` |
| DB migrations | `[pnpm prisma migrate dev --name <change>]` |

A change is not done until type-check, lint, and tests for touched files pass. Run `pnpm build` before handing off a change to startup, config, or middleware order.

## Layout

- `src/app.ts` — builds the Express app (middleware, routers, error handler); no `listen()` here so tests can import it
- `src/server.ts` — reads config, calls `app.listen()`, handles shutdown signals
- `src/routes/` — one router per resource (`users.routes.ts`), mounted in `app.ts`
- `src/controllers/` or handlers — parse request, call a service, shape the response
- `src/services/` — business logic; no `req` / `res` in here
- `src/middleware/` — auth, validation, request logging, error handling
- `src/config.ts` — the only place that reads `process.env`; validated at startup
- `src/db/` or `prisma/` — schema, migrations, client
- `test/` or `*.test.ts` next to code — supertest against `app`
- Do not edit `dist/`, `node_modules/`, or generated clients

## Conventions

1. **Follow the existing structure.** Match current router, controller, and service patterns. Don't introduce a new framework layer, DI container, ORM, or logger unless the repo already uses it.
2. **Thin handlers.** Handlers validate input, call a service, and send one response. Business rules live in services so they can be tested without HTTP.
3. **Validate at the edge.** `req.body`, `req.params`, and `req.query` are untrusted. Parse them with [zod] (or the repo's validator) before use and return `400` with field errors. Never pass raw `req.body` into a DB call.
4. **Async errors reach the error handler.** On Express 4, wrap async handlers (or use the repo's `asyncHandler`) so a rejected promise calls `next(err)`. Express 5 forwards rejected promises itself — don't add redundant wrappers there. Never leave a promise unhandled.
5. **One error handler, registered last.** A single `(err, req, res, next)` middleware maps known errors to status codes and returns a consistent JSON shape. Don't leak stack traces or internal messages in production responses.
6. **Send exactly one response.** `return` after `res.status(...).json(...)` / `next(err)` so code doesn't fall through and trigger "headers already sent".
7. **Middleware order matters.** Security headers, CORS, body parsing, and auth come before routers; 404 then error handler come last. Ask before reordering.
8. **Config from one place.** Read `process.env` only in `src/config.ts`, validate it at startup, and fail fast if something required is missing. Document names in `.env.example`. Never read, print, or commit `.env*` secrets.
9. **Security basics.** Keep [helmet], explicit CORS origins (no `*` with credentials), body size limits, and rate limiting on auth routes. Parameterized queries only; never build SQL from strings.
10. **TypeScript strict.** Type request bodies and responses via validator inference. No `any` (use `unknown` + narrow); avoid `!` assertions without a one-line why. Augment `Express.Request` in a `.d.ts` for things like `req.user` instead of casting.
11. **Logging.** Use the repo's logger ([pino / winston]) with a request id. No `console.log` left in handlers; never log passwords, tokens, cookies, or full request bodies.
12. **Dependencies.** Ask before adding an npm package or middleware. Prefer libraries already in the tree.

## Testing

- supertest against the exported `app` (not a running server) for routes, status codes, validation errors, and auth.
- Unit tests for services with the DB or external APIs faked at the boundary.
- New behavior gets a test; a bug fix gets a test that fails before the fix.

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by refactors or reformatting of untouched files
- [ ] Every new route validates params, query, and body and returns `400` on bad input
- [ ] Async errors reach the error handler; each path sends exactly one response
- [ ] No secrets, stack traces, or PII in responses or logs; `process.env` only read in config
- [ ] Auth and authorization checked on new routes (not just "logged in", but allowed to touch this record)
- [ ] Migrations included for schema changes, and they're reversible or noted
- [ ] Type-check, lint, relevant tests, and `pnpm build` (for startup/middleware changes) pass
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify)

## Boundaries — ask first

- Adding a dependency, changing middleware order, CORS, auth, session or cookie settings, CI, or the database schema
- Upgrading Express major versions or switching module systems (CommonJS ↔ ESM)
- Deleting endpoints other clients may call, rewriting git history, force-pushing, or touching production infra or deploy config

## Never

- Edit `dist/` or generated clients, hand-edit lockfiles, or commit generated noise
- Concatenate user input into SQL, shell commands, file paths, or redirects
- Disable lint/type rules project-wide to "make it green"
- Log tokens, cookies, or PII, or claim a task is done when type-check, tests, or the build failed

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
