# AGENTS.md — Django (Python)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** Django [5.x], Python [3.11+], optional Django REST Framework, package tool [pip / uv / poetry], tests via [pytest-django / manage.py test]. Replace bracketed defaults if your project differs.

## Project

- App: [PROJECT NAME] — [one-line purpose]
- Runtime: Python [3.11+] · Django [5.x] · [DRF yes/no]
- Database: [PostgreSQL / SQLite for local]
- Settings module: `[config.settings.dev]` (prod: `[config.settings.prod]`)

## Commands (repo root)

| Task | Command |
|------|---------|
| Install | `[pip install -r requirements.txt / uv sync / poetry install]` |
| Dev server | `python manage.py runserver` |
| Make migrations | `python manage.py makemigrations [app]` |
| Apply migrations | `python manage.py migrate` |
| Migration check | `python manage.py makemigrations --check --dry-run` |
| Tests | `[pytest / python manage.py test]` · one file: `[pytest path/to/test_x.py]` |
| Lint / format | `[ruff check . && ruff format --check .]` |
| System checks | `python manage.py check --deploy` (prod settings) |

A change is not done until tests and lint for touched apps pass and the migration check is clean. Iterate on the narrowest test path; run the full suite before handing off.

## Layout

- `config/` (or `[project]/`) — settings, root `urls.py`, `wsgi.py` / `asgi.py`
- `[app]/models.py` — models; one app per domain area
- `[app]/views.py` / `[app]/api/` — views or DRF viewsets
- `[app]/serializers.py` / `forms.py` — input validation at the edge
- `[app]/services.py` — business logic that spans models (keep views thin)
- `[app]/migrations/` — generated migrations, committed with the model change
- `[app]/tests/` — tests next to the app they cover
- `templates/`, `static/` — shared templates and assets

## Conventions

1. **Apps own domains.** New features go in the app that owns the data. Avoid circular imports between apps; reference models by `"app.Model"` strings in relations when needed.
2. **Migrations.** Every model change ships with its generated migration in the same diff. Never edit an applied migration; add a new one. Data migrations use `RunPython` with a reverse function.
3. **Queries.** Use the ORM. Watch for N+1 queries — reach for `select_related` / `prefetch_related` in list views. Raw SQL only with parameters, never string formatting.
4. **Validation.** Validate input with forms or DRF serializers, not by hand in views. Model `clean()` for invariants that must hold everywhere.
5. **Settings & secrets.** Read secrets from environment variables. Never commit `SECRET_KEY`, DB passwords or API keys. `DEBUG = False` in prod settings.
6. **Security defaults.** Keep CSRF middleware on; don't add `@csrf_exempt` without a one-line why. Use `django.contrib.auth` permissions/decorators (or DRF permission classes) on every new view that touches user data.
7. **Dependencies.** Ask before adding a package. Prefer what is already in the tree and update the lock/requirements file through the project's tool.

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by refactors
- [ ] Model changes include a migration; `makemigrations --check` is clean
- [ ] New views have auth/permission checks matching the app's existing pattern
- [ ] No obvious N+1 in new list endpoints or templates
- [ ] No secrets, tokens, or PII in logs, fixtures, or commits
- [ ] Tests and lint pass on touched apps
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify)

## Boundaries — ask first

- Adding a dependency, changing Django / Python major version, CI, auth backends, or custom user model
- Destructive migrations (dropping columns/tables), squashing migrations, or touching production data
- Rewriting git history, force-pushing, or changing deploy infra

## Never

- Commit `.env`, real `SECRET_KEY`s, local `db.sqlite3`, or `__pycache__/`
- Edit or delete migrations that have already run in shared environments
- Turn on `DEBUG` or widen `ALLOWED_HOSTS` to `*` in prod settings
- Claim a task is done when tests, lint, or the migration check failed

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
