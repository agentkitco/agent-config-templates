# AGENTS.md — Laravel (PHP)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** Laravel [11.x / 12.x], PHP [8.2+], Composer, tests via [PHPUnit / Pest], lint via [Pint]. Frontend optional: [Blade / Livewire / Inertia]. Replace bracketed defaults if your project differs.

## Project

- App: [PROJECT NAME] — [one-line purpose]
- Runtime: PHP [8.2+] · Laravel [11.x / 12.x] · [API-only yes/no]
- Database: [PostgreSQL / MySQL / SQLite for local]
- Frontend: [Blade / Livewire / Inertia (React or Vue) / none]
- Queue / cache: [database / Redis / sync] · Horizon: [yes/no]

## Commands (repo root)

| Task | Command |
|------|---------|
| Install | `composer install` |
| Env | `cp .env.example .env` then `php artisan key:generate` |
| Dev server | `php artisan serve` (or [Vite + `npm run dev`] alongside) |
| Generate migration | `php artisan make:migration [create_x_table]` |
| Apply migrations | `php artisan migrate` |
| Tests | `[php artisan test / ./vendor/bin/pest]` · one file: `[php artisan test --filter=XTest]` |
| Lint / format | `./vendor/bin/pint` |
| Clear caches | `php artisan optimize:clear` |

A change is not done until tests and Pint for touched files pass and migrations are applied cleanly. Iterate on the narrowest test; run the full suite before handing off.

## Layout

- `app/Models/` — Eloquent models, casts, relationships, scopes
- `app/Http/Controllers/` — thin controllers; heavy logic in Actions / Services if the repo already uses them
- `app/Http/Requests/` — Form Request classes (validation + authorization)
- `app/Policies/` — authorization policies bound to models
- `routes/web.php`, `routes/api.php`, `routes/console.php` — route definitions
- `database/migrations/` — migrations, committed with the model change
- `resources/views/` — Blade templates (or Inertia pages under `resources/js/`)
- `tests/Feature/`, `tests/Unit/` — tests mirroring app behavior

## Conventions

1. **Laravel way first.** Prefer Artisan generators and existing patterns. Don't introduce a new architecture (Actions, DTOs, repositories) unless the codebase already uses it.
2. **Migrations.** Every schema change ships with its migration in the same diff. Never edit a migration that has run in a shared environment; add a new one. Make migrations reversible (`up`/`down` or safe `Schema` changes).
3. **Queries.** Watch for N+1 — use `with()` / eager loading on list endpoints and views. Use query builder / Eloquent bindings; never interpolate user input into raw SQL.
4. **Validation.** Prefer Form Requests (`php artisan make:request`) over inline `$request->validate()` for non-trivial create/update. Model `$fillable` / `$guarded` must stay intentional — no mass-assignment of secrets or role fields.
5. **Auth.** Every new controller action that touches user data uses the app's existing guard, middleware, and policies ([Sanctum / Passport / Breeze / Fortify / Jetstream]). Call `$this->authorize(...)` or route middleware where peers do.
6. **Secrets.** Keep secrets in `.env` (and config that reads `env()`). Never commit `.env`, `storage/oauth-*.key`, or real API keys.
7. **Dependencies.** Ask before adding a Composer or npm package. Update lockfiles only through `composer` / `npm`.

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by refactors
- [ ] Schema changes include a reversible migration
- [ ] Form Requests / validation and auth checks match existing controllers
- [ ] No obvious N+1 in new index actions, resources, or Blade/Inertia pages
- [ ] No secrets, tokens, or PII in logs, factories, seeders, or commits
- [ ] Tests and Pint pass on touched files
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify)

## Boundaries — ask first

- Adding a Composer/npm package, upgrading Laravel or PHP, changing CI, auth, or the queue/cache backend
- Destructive migrations (dropping columns/tables), long-locking migrations on big tables, or touching production data
- Rewriting git history, force-pushing, or changing deploy config (Forge, Vapor, Envoyer, Docker, etc.)

## Never

- Commit `.env`, `storage/*.key`, compiled views caches you didn't mean to, or local database files
- Edit migrations that already ran in shared environments
- Disable CSRF (`VerifyCsrfToken` exceptions) or use `$request->all()` for mass assignment without a one-line why
- Claim a task is done when tests or Pint failed

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
