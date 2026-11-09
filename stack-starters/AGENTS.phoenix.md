# AGENTS.md — Phoenix (Elixir)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** Elixir [1.17+] / OTP [27], Phoenix [1.7 / 1.8], Ecto with [PostgreSQL], LiveView [1.x], assets via [esbuild + Tailwind], tests with ExUnit. Replace bracketed defaults if your project differs. Check the current Phoenix and Ecto docs for your version, since generators and APIs change between releases.

## Project

- App: [PROJECT NAME] — [one-line purpose]
- OTP app name: `[my_app]` · web module: `[MyAppWeb]`
- Database: [PostgreSQL via `Ecto.Adapters.Postgres`]
- Background jobs: [Oban / none] · auth: [`phx.gen.auth` / other]

## Commands (repo root)

| Task | Command |
|------|---------|
| Install deps | `mix deps.get` |
| Set up DB + assets | `mix setup` |
| Dev server | `mix phx.server` (or `iex -S mix phx.server`) → http://localhost:4000 |
| Compile (strict) | `mix compile --warnings-as-errors` |
| Format | `mix format` (check: `mix format --check-formatted`) |
| Lint | `[mix credo --strict]` |
| Tests | `mix test` (one file: `mix test test/path_test.exs:42`) |
| New migration | `mix ecto.gen.migration add_[thing]` |
| Migrate | `mix ecto.migrate` (rollback: `mix ecto.rollback`) |
| Pre-commit bundle | `[mix precommit]` if the repo defines it |
| Deploy | `[your release / host command]` (ask first) |

A change is not done until it compiles without warnings, `mix format --check-formatted` passes, and `mix test` is green.

## Layout

- `lib/[my_app]/` — business logic in **contexts** (e.g. `Accounts`, `Catalog`): schemas, changesets, queries
- `lib/[my_app]_web/` — web layer: router, controllers, LiveViews, components, plugs
- `lib/[my_app]_web/components/core_components.ex` — shared function components; reuse before writing new ones
- `priv/repo/migrations/` — Ecto migrations · `priv/repo/seeds.exs` — seeds
- `config/` — `config.exs`, `dev.exs`, `test.exs`, `runtime.exs` (production secrets come from env here)
- `assets/` — JS, CSS, Tailwind config · `test/` — mirrors `lib/`, with `test/support/` for `ConnCase`, `DataCase`, fixtures
- Do not edit `_build/`, `deps/`, `priv/static/assets/`, or generated digests

## Conventions

1. **Follow the existing structure.** Match current contexts, naming, and generator output. Don't add a new library, auth system, or job runner the repo doesn't already use.
2. **Contexts own the data.** Controllers and LiveViews call context functions (`Accounts.get_user!/1`, `Catalog.create_product/1`); they don't call `Repo` directly or build Ecto queries.
3. **Changesets validate everything.** All writes go through a changeset with explicit `cast/3` fields and `validate_*` calls. Never cast fields a user shouldn't set (roles, owner ids, prices); set those in code.
4. **Return tagged tuples.** Context writes return `{:ok, struct}` / `{:error, changeset}`; handle both with `case` or `with`. Use bang functions (`get!`) only where a crash (404) is the right outcome.
5. **Scope data to the current user.** Queries for user-owned records filter by the owner (or the repo's scope struct). Never trust an id from params alone.
6. **Avoid N+1 queries.** Preload associations in the context (`Repo.preload` or `preload:` in the query) instead of loading them per row in a template.
7. **Migrations are additive and reversible.** New migration files only; never edit one that has run in shared or production environments. Add indexes for foreign keys and lookup columns. Ask before dropping or renaming columns.
8. **LiveView state stays small.** Keep assigns minimal, use `stream/3` for long lists, do expensive work in `handle_async` or `assign_async`, and only subscribe to PubSub when `connected?(socket)`. Validate every `handle_event` payload; it comes from the client.
9. **Use function components and verified routes.** Reuse `core_components` (`<.input>`, `<.button>`, `<.table>`), and write paths with `~p"/..."` so broken routes fail at compile time.
10. **HEEx escapes by default.** Never use `raw/1` on user content. Keep CSRF protection in the `:browser` pipeline.
11. **Config and secrets.** Production values are read in `config/runtime.exs` from environment variables. Never commit secrets or put them in `config.exs` / `prod.exs`.
12. **Let it crash, but not silently.** Don't wrap everything in `try/rescue`. Supervise long-running processes; log with `Logger` and useful metadata.
13. **Dependencies.** Ask before adding a Hex package or changing `mix.exs` versions.

## Testing

- Context tests use `DataCase`; controller and LiveView tests use `ConnCase` with `Phoenix.LiveViewTest` (`live/2`, `render_click/2`, `render_submit/2`).
- Use fixtures in `test/support/fixtures/` (or the repo's factory library) rather than inserting raw rows.
- Tests run in the SQL sandbox; mark `async: true` when the module doesn't share global state.
- New behavior gets a test; a bug fix gets a test that fails before the fix where practical.

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by refactors or reformatting of untouched files
- [ ] `mix compile --warnings-as-errors`, `mix format --check-formatted`, and `mix test` pass
- [ ] Web layer calls contexts; no `Repo` calls in controllers or LiveViews
- [ ] Changesets cast only safe fields; user-owned queries are scoped to the current user
- [ ] New queries preload what templates use; new lookup columns have indexes
- [ ] Migrations are new files and reversible; no `raw/1` on user content; no secrets committed
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify)

## Boundaries — ask first

- Adding a dependency, changing Elixir/OTP or Phoenix versions, or changing the auth setup
- Destructive migrations (dropping or renaming columns or tables) or running migrations against shared databases
- Deploying, building releases for production, or changing production environment variables

## How to customize

1. Fill `[BRACKETED]` values and delete options you don't use (LiveView, Oban, Credo).
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
