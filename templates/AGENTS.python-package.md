# AGENTS.md

## Project
[PROJECT NAME]: [ONE LINE]. Python [3.XX]. Environment: [uv / venv + pip / poetry].

## Commands
- Set up: `[uv sync]` or `[python -m venv .venv && . .venv/bin/activate && pip install -e ".[dev]"]`
- Tests: `[pytest -q]`; one test: `[pytest path/to/test_file.py::test_name -q]`
- Lint and format check: `[ruff check .]` and `[ruff format --check .]`
- Types: `[mypy src]` (only if configured)
Tests, lint and format check must pass before a task is done.

## Layout
- `src/[package]/` code. `tests/` mirrors `src/` (module `a/b.py` is tested in `tests/a/test_b.py`). `scripts/` one-off tools.

## Conventions
- Type hints on every public function. Prefer `pathlib.Path` over string paths.
- Raise specific exceptions; never use a bare `except:`.
- No print debugging in library code: use `logging`.
- Tests: pytest style (plain `assert`), one behaviour per test, fixtures in `conftest.py`. No network or real clock in unit tests; patch them.
- New dependencies go in `[pyproject.toml]`, not ad hoc `pip install`.

## Boundaries
- Ask before adding a dependency or changing packaging config.
- Don't touch `.venv/`, `build/`, `dist/` or `*.egg-info`.
- Never run scripts in `scripts/` that write outside the repo without asking.
