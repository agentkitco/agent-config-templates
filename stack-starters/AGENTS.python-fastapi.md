# AGENTS.md — Python FastAPI

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** FastAPI + Python 3.11+, pydantic v2, uvicorn, pytest, ruff. Replace bracketed defaults if your project differs.

## Project

- App: [PROJECT NAME] — [one-line purpose]
- Runtime: Python [3.11+] · Env manager: [uv]
- Framework: FastAPI + pydantic v2

## Commands (repo root)

| Task | Command |
|------|---------|
| Install / sync | `uv sync` (or `python -m venv .venv && source .venv/bin/activate && pip install -e ".[dev]"`) |
| Dev server | `uv run uvicorn [app.main:app] --reload --port 8000` |
| Tests | `uv run pytest -q` · one test: `uv run pytest path/to/test_file.py::test_name -q` |
| Lint | `uv run ruff check .` |
| Format check | `uv run ruff format --check .` |
| Types (if configured) | `uv run mypy [src]` |

A change is not done until lint, format check, and tests for touched modules pass. Prefer the narrowest pytest selection while iterating.

## Layout

- `src/[package]/` or `[app]/` — application code
  - `main.py` — FastAPI app factory / entry
  - `api/` or `routers/` — route modules
  - `schemas/` — pydantic request/response models
  - `services/` — business logic (no FastAPI imports when avoidable)
  - `deps.py` — shared `Depends()` callables
- `tests/` — mirrors package layout (`test_*.py`)
- `scripts/` — one-off tools (ask before anything that writes outside the repo)

## Conventions

1. **Type hints** on every public function and endpoint. Prefer `pathlib.Path` over string paths.
2. **pydantic models** for every request body, query bundle you care about, and response shape. Avoid raw `dict` at the HTTP boundary.
3. **Routers stay thin.** Validate → call a service → return a schema. Keep DB/session and external I/O in services or repositories.
4. **Dependencies** via `Depends()` for auth, DB sessions, and settings — do not construct globals inside route functions.
5. **Errors.** Raise `HTTPException` (or domain exceptions mapped in handlers) with safe detail strings. Never bare `except:`.
6. **Settings.** Load env through a pydantic `BaseSettings` (or equivalent). Document names in `.env.example`. Never print or commit secrets.
7. **Logging.** Use `logging`, not `print`, in library and route code.
8. **Tests.** pytest + `httpx.AsyncClient` (or TestClient) for API tests. No real network or wall clock in unit tests — patch them. One behaviour per test.

## Review habits (before you say “done”)

- [ ] Diff is scoped to the ask — no drive-by refactors
- [ ] New endpoints have request/response models and status codes that match the OpenAPI intent
- [ ] Auth/permission checks sit on protected routes (not “we’ll add later”)
- [ ] No secrets in logs, fixtures, or commits
- [ ] ruff + relevant pytest pass
- [ ] If you opened a PR: blockers-first summary (what broke if wrong → what changed → how to verify, including curl/httpie examples for new routes)

## Boundaries — ask first

- Adding a dependency, changing packaging (`pyproject.toml`), CI, auth, or database migrations/schema
- Running destructive scripts, dropping databases, or calling paid third-party APIs
- Deleting public modules or rewriting git history

## Never

- Touch `.venv/`, `build/`, `dist/`, or `*.egg-info`
- Commit `.env*` secrets or real API keys in tests
- Disable ruff/mypy project-wide to “make it green”
- Claim a task is done when checks failed

## How to customize

1. Fill `[BRACKETED]` values (module path for uvicorn, package name, Python version).
2. Delete sections that do not apply — keep the file short.
3. Optionally add a one-line `CLAUDE.md` with `@AGENTS.md` so Claude Code shares this file.

---

### Disclaimer

Written with AI assistance by [AgentKit Co](https://agentkitco.com/). Starting point, not a guarantee — review against your repo and keep real enforcement in CI.

### Want more?

- Free templates + generator: https://agentkitco.com/tools/agents-md-generator/
- [AI Coding Agent Config Kit](https://agentkitco.com/products/agent-config-kit/) ($14) — 12-stack AGENTS.md set, Cursor rules, Copilot files
- [AI PR Review Kit](https://agentkitco.com/products/ai-pr-review-kit/) ($17) — blockers-first checklists, paste-ready PR comments, review prompts for Claude Code & Cursor

License: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — use and adapt with attribution to AgentKit Co.
