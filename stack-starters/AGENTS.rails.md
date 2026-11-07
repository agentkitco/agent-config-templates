# AGENTS.md — Ruby on Rails

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** Rails [7.x / 8.x], Ruby [3.2+], Bundler, tests via [RSpec / Minitest], lint via [RuboCop / standardrb]. Replace bracketed defaults if your project differs.

## Project

- App: [PROJECT NAME] — [one-line purpose]
- Runtime: Ruby [3.2+] · Rails [7.x / 8.x] · [API-only yes/no]
- Database: [PostgreSQL / MySQL / SQLite for local]
- Frontend: [Hotwire (Turbo + Stimulus) / React via jsbundling / none]
- Background jobs: [Solid Queue / Sidekiq / none]

## Commands (repo root)

| Task | Command |
|------|---------|
| Install | `bundle install` |
| Setup | `bin/setup` |
| Dev server | `bin/dev` (or `bin/rails server`) |
| Generate migration | `bin/rails generate migration [AddXToY]` |
| Apply migrations | `bin/rails db:migrate` |
| Tests | `[bundle exec rspec / bin/rails test]` · one file: `[bundle exec rspec spec/models/x_spec.rb]` |
| Lint | `[bundle exec rubocop / bundle exec standardrb]` |
| Security scan | `[bin/brakeman]` |

A change is not done until tests and lint for touched files pass and `db/schema.rb` (or `structure.sql`) matches the migrations. Iterate on the narrowest spec; run the full suite before handing off.

## Layout

- `app/models/` — ActiveRecord models, validations, scopes
- `app/controllers/` — thin controllers; strong params live here
- `app/views/`, `app/components/` — templates / view components
- `app/jobs/` — background jobs (idempotent, safe to retry)
- `app/services/` (or `app/models/concerns/`) — business logic spanning models
- `db/migrate/` — migrations, committed with the model change; `db/schema.rb` regenerated, not hand-edited
- `config/routes.rb` — routes; prefer `resources` over custom routes
- `spec/` or `test/` — tests mirroring `app/`

## Conventions

1. **Rails way first.** Follow the existing generators and naming. Don't introduce a new pattern (service objects, form objects, interactors) unless the codebase already uses it.
2. **Migrations.** Every schema change ships with its migration and the regenerated schema file in the same diff. Never edit a migration that has run in a shared environment; add a new one. Make migrations reversible (`change` or `up`/`down`).
3. **Queries.** Watch for N+1 — use `includes` / `preload` in list views. Use parameterized `where` (hash or `?` placeholders); never interpolate user input into SQL.
4. **Params & validation.** Use strong params (`params.require(...).permit(...)`) in every create/update action. Model validations for invariants; add a DB constraint for anything that must never break.
5. **Auth.** Every new controller action that touches user data uses the app's existing authentication and authorization pattern ([Devise / Rails 8 auth generator / Pundit / CanCanCan]).
6. **Secrets.** Use Rails credentials or environment variables. Never commit `config/master.key`, `.env`, or real API keys.
7. **Dependencies.** Ask before adding a gem. Update `Gemfile.lock` only through `bundle`.

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by refactors
- [ ] Schema changes include a reversible migration and the regenerated schema file
- [ ] Strong params and auth checks match existing controllers
- [ ] No obvious N+1 in new index actions, views, or serializers
- [ ] No secrets, tokens, or PII in logs, fixtures, or commits
- [ ] Tests, lint, and (if used) Brakeman pass on touched files
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify)

## Boundaries — ask first

- Adding a gem, upgrading Rails or Ruby, changing CI, auth, or the job backend
- Destructive migrations (dropping columns/tables), long-locking migrations on big tables, or touching production data
- Rewriting git history, force-pushing, or changing deploy config (Kamal, Capistrano, Heroku, etc.)

## Never

- Commit `config/master.key`, `.env`, `log/`, `tmp/`, or local database files
- Hand-edit `db/schema.rb` or edit migrations that already ran in shared environments
- Use `skip_before_action :verify_authenticity_token` or `permit!` without a one-line why
- Claim a task is done when tests or lint failed

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
