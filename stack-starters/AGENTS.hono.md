# AGENTS.md — Hono (TypeScript)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** Hono [4.x], TypeScript strict, package manager [pnpm], validation with [`@hono/zod-validator`](https://github.com/honojs/middleware), runtime [Cloudflare Workers / Node.js via `@hono/node-server` / Deno / Bun]. Replace bracketed defaults if your project differs. If the repo is plain JavaScript, match it — don't convert files to TypeScript unless asked.

## Project

- Service: [PROJECT NAME] — [one-line purpose]
- Runtime: [Cloudflare Workers] · Hono [4.x] · TypeScript (strict)
- Package manager: [pnpm]
- Data: [D1 / KV / R2 / Postgres via Hyperdrive / none] · auth: [JWT / session / none]

## Commands (repo root)

| Task | Command |
|------|---------|
| Install | `pnpm install` |
| Dev | `[pnpm dev]` (Workers: `wrangler dev` → http://localhost:8787; Node: `tsx watch src/index.ts` → http://localhost:[3000]) |
| Type-check | `pnpm typecheck` (`tsc --noEmit`) |
| Lint / format | `pnpm lint` · `pnpm format` |
| Tests | `pnpm test` · one file: `[pnpm vitest run path/to/file.test.ts]` |
| Worker types | `[wrangler types]` (regenerate after `wrangler.jsonc` / binding changes) |
| Deploy (Workers) | `[wrangler deploy]` (ask first) |
| Node production | `[node dist/index.js]` after `pnpm build` |

A change is not done until type-check, lint, and tests for touched files pass. After changing bindings or `wrangler.jsonc`, regenerate types before handing off.

## Layout

- `src/index.ts` — builds the Hono app, mounts middleware and routes, exports `app` (and `type AppType = typeof app` if the repo uses `hono/client`)
- `src/routes/` — one Hono instance per resource (`posts.ts`), mounted with `app.route('/posts', posts)`
- `src/services/` — business logic; no `Context` / `c` in here so it can be tested without HTTP
- `src/middleware/` — auth, request ids, admin gates
- `src/validators/` — zod (or the repo's validator) schemas shared by routes and tests
- `src/lib/` — typed helpers; keep them runtime-safe (Web Standard APIs)
- `[wrangler.jsonc]` / `[wrangler.toml]` — Worker name, compatibility date, bindings (Workers projects)
- Do not edit `dist/`, `.wrangler/`, `node_modules/`, or generated `worker-configuration.d.ts`

On Cloudflare Workers, `export default app` (or `{ fetch: app.fetch }`). On Node, keep `listen()` / `serve()` in a separate entry so tests can import `app` without opening a port.

## Conventions

1. **Follow the existing structure.** Match current routing, middleware, and folder patterns. Don't introduce Express-style `req`/`res`, a new validator, ORM, or logger unless the repo already uses it.
2. **Stay on Web Standard APIs.** Hono runs on Workers, Deno, Bun, and Node. Prefer `fetch`, `Request`, `Response`, `Headers`, and `c.env` bindings. Ask before using Node-only APIs (`fs`, `process.env`, `node:http`) — they break Workers unless `nodejs_compat` is on and the repo already relies on them.
3. **Thin handlers.** Handlers read validated input, call a service, and `return c.json(...)` / `c.text(...)` / `c.html(...)`. Always `return` the response so execution does not fall through.
4. **Validate at the edge.** Use `[zValidator('json' | 'query' | 'param' | 'form', schema)]` (or the repo's validator) and then `c.req.valid('json')`. Never pass raw `c.req.json()` / query strings into a database call. Return `400` with field errors on bad input.
5. **Read the body once.** The request body can only be consumed once. If a validator already parsed it, use `c.req.valid(...)` — don't call `c.req.json()` again.
6. **Type Bindings and Variables.** Parameterize the app: `new Hono<{ Bindings: Env; Variables: { user: User } }>()`. Use `c.set('user', user)` / `c.get('user')` instead of untyped casts. On Workers, treat `c.env` (and `wrangler types`) as the source of truth for secrets and bindings — not `process.env`.
7. **One `onError` and one `notFound`.** `app.onError` maps known errors to status codes and a consistent JSON shape. `app.notFound` covers unknown routes. Don't leak stack traces or internal messages in production responses.
8. **Middleware order matters.** Logger, `secureHeaders()`, CORS, and auth come before routed handlers. Ask before reordering or widening CORS (no `origin: '*'` with credentials).
9. **RPC types stay in sync.** If the repo exports `AppType` for `hc<AppType>()` from `hono/client`, add new routes on that same typed chain so the client keeps compiling. Don't break exported route types silently.
10. **Bindings and secrets.** Document binding names next to `wrangler.jsonc`. Put secrets in `[wrangler secret]` / the dashboard, never in source or committed `.dev.vars` that contain real values. Commit `.dev.vars.example` only.
11. **Logging.** Use the repo's logger (or `c.var` request id). No `console.log` left in handlers; never log tokens, cookies, secrets from `c.env`, or full request bodies.
12. **Dependencies.** Ask before adding an npm package, Hono middleware, or a new runtime adapter. Prefer middleware already in the tree (`hono/cors`, `hono/secure-headers`, `@hono/zod-validator`).

## Testing

- Call `app.request('/path', { method, headers, body })` (or `app.request(new Request(url, init))`) — no listening server and no supertest.
- For Workers bindings, pass a stub `env` via `app.request(input, init, env)` (or the repo's helper) so tests don't need a live KV/D1/R2 account.
- Unit-test services with external APIs and bindings faked at the boundary.
- New behavior gets a test; a bug fix gets a test that fails before the fix.

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by refactors or reformatting of untouched files
- [ ] Every new route validates params, query, and body and returns `400` on bad input
- [ ] Handlers `return` exactly one response; `onError` covers thrown errors
- [ ] No Node-only APIs added to a Workers/Deno/Bun target; no secrets, stack traces, or PII in responses or logs
- [ ] Auth and authorization checked on new routes (not just "logged in", but allowed to touch this record)
- [ ] New or changed bindings are reflected in `wrangler.jsonc` and regenerated types
- [ ] Exported `AppType` / RPC client still type-checks if the repo uses `hono/client`
- [ ] Type-check, lint, and relevant tests pass
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify)

## Boundaries — ask first

- Adding a dependency, changing middleware order, CORS, auth, cookie settings, CI, or bindings
- Switching runtimes (Workers ↔ Node ↔ Bun ↔ Deno) or enabling `nodejs_compat`
- Deploying (`wrangler deploy`), changing production secrets, or pointing bindings at a different store
- Deleting endpoints other clients (including `hc<AppType>()`) may call, rewriting git history, force-pushing, or touching production infra

## Never

- Use Express `req`/`res`/`next` APIs, or assume `process.env` exists on Workers
- Concatenate user input into SQL, KV keys used as path prefixes without validation, shell commands, file paths, or redirects
- Commit `.dev.vars` secrets, wrangler state, or real API tokens
- Disable lint/type rules project-wide to "make it green"
- Log tokens, cookies, or PII, or claim a task is done when type-check or tests failed

## How to customize

1. Fill `[BRACKETED]` values and delete options you don't use (Workers vs Node commands, especially).
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
