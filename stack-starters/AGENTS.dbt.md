# AGENTS.md — dbt (analytics engineering, SQL)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** dbt [Core 1.8+ / dbt Cloud / Fusion] on [Snowflake / BigQuery / Databricks / Postgres / Redshift / DuckDB], packages in `packages.yml` (for example [dbt_utils]), SQL linted with [SQLFluff], and CI that builds changed models with state comparison ([`dbt build --select state:modified+`]). Replace bracketed defaults if your project differs, and check the current dbt and adapter docs for your versions, since config keys and defaults change between releases.

## Project

- Repo: [PROJECT NAME] — [what data it models and who uses it]
- Warehouse: [adapter] · dev target: [`dev`, one schema per developer] · prod target: [`prod`, run only by the scheduler]
- Sources: [raw schemas / loaders, e.g. Fivetran, Airbyte, custom ELT]
- Consumers: [BI tool / dashboards / reverse ETL / ML] — downstream exposures in [`models/exposures.yml`]

## Commands (dev target only)

| Task | Command |
|------|---------|
| Install packages | `dbt deps` |
| Check connection | `dbt debug` |
| Compile (no warehouse writes) | `dbt compile --select [model]` |
| Build one model + its tests | `dbt build --select [model]` |
| Build a model and everything downstream | `dbt build --select [model]+` |
| Tests only | `dbt test --select [model]` |
| Source freshness | `dbt source freshness` |
| Lint / fix SQL | `[sqlfluff lint models/]` / `[sqlfluff fix models/]` |
| Prod runs | **never run by the agent** — the scheduler/CI runs prod |

A change is not done until the touched models and their downstream children build and test cleanly on the dev target. Always pass `--select`; don't run the whole project unless asked, since it costs warehouse credits. If you can't connect to the warehouse, say so plainly and run `dbt compile` / `dbt parse` instead.

## Layout

- `models/staging/<source>/` — `stg_<source>__<table>.sql`: one model per source table, rename, cast and light cleanup only, materialized as views
- `models/intermediate/` — `int_<entity>_<verb>.sql`: joins and reshaping that aren't exposed to users
- `models/marts/<domain>/` — `fct_<event>.sql` and `dim_<entity>.sql`: what BI and consumers read
- `models/**/_<dir>__models.yml` and `_<source>__sources.yml` — descriptions, tests, contracts
- `macros/`, `seeds/` (small static CSVs only), `snapshots/`, `tests/` (singular tests), `analyses/`
- Never edit or commit `target/`, `dbt_packages/`, `logs/` or `profiles.yml` with credentials.

## Conventions

1. **Follow the existing layers and naming.** Staging reads only from `{{ source() }}`; everything else reads from `{{ ref() }}`. Never hard-code `database.schema.table` names.
2. **One grain per model.** State the grain (one row per ...) in the model description and protect it with `unique` + `not_null` tests on the primary key.
3. **Test what you touch.** Every new or changed model has a primary-key test, `relationships` tests for foreign keys, and `accepted_values` for status-style columns. Add a singular test in `tests/` for business rules that must hold.
4. **Document columns.** New columns get a description in the YAML file next to the model; keep model contracts (`contract: {enforced: true}`) in sync when they exist.
5. **Readable SQL.** Import CTEs at the top (`with orders as (select * from {{ ref('stg_shop__orders') }})`), then logical CTEs, then a final `select`. Explicit column lists in marts; no `select *` in the final select. Lowercase keywords and trailing commas if that's the house style ([SQLFluff config]).
6. **Joins on purpose.** Say why a join is `left` vs `inner`, and check for fan-out: row counts before and after a join should match the grain you intended.
7. **Incremental models carefully.** Keep `unique_key`, the `is_incremental()` filter and `on_schema_change` as they are unless the ask is about them. Changing an incremental model's logic usually needs a `--full-refresh`, which is an ask-first.
8. **Don't break consumers.** Renaming or dropping a column in a mart, changing a model's grain or materialization, or deleting a model is a breaking change: check `ref`s, exposures and the BI layer, and call it out at the top of the PR.
9. **Macros are a last resort.** Prefer plain SQL or an existing package macro ([`dbt_utils`]) over a new Jinja macro. Keep Jinja shallow and compiled SQL readable.
10. **No PII leaks.** Don't move raw email, phone, address or payment fields into marts unless the ask says so; follow the existing masking or hashing pattern.
11. **Cost awareness.** Don't switch views to tables, add clustering/partitioning, or schedule frequent full refreshes without saying what it costs and why.

## Testing

- `dbt build --select [changed models]+` on dev: models build and all tests pass.
- For logic changes, compare a few key numbers (row counts, totals for a recent period) between prod and your dev build and paste them in the PR; explain any difference.
- If CI uses Slim CI (`state:modified+` with `--defer`), make sure your change passes there, not just locally.

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by renames of models or columns
- [ ] Only `ref()` / `source()`, no hard-coded relations
- [ ] Grain stated and protected by `unique` + `not_null`; new columns documented
- [ ] Touched models and downstream children build and test on dev; selection used, not a full-project run
- [ ] No fan-out from joins; row counts checked against the intended grain
- [ ] Breaking changes to marts, exposures or contracts called out at the top of the PR
- [ ] SQLFluff passes; no credentials or `target/` files committed
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify)

## Boundaries — ask first

- Running anything against the prod target, `--full-refresh` on large or incremental models, or `dbt run-operation` macros that write or drop
- Dropping, renaming or changing the grain of a mart, an exposure or a contracted model
- Changing `dbt_project.yml` materialization defaults, warehouse sizing, grants or `profiles.yml`
- Adding or upgrading packages, or upgrading dbt / the adapter
- Exposing new PII columns or changing masking/access policies

## How to customize

1. Fill `[BRACKETED]` values and delete options you don't use (warehouse, linter, CI style).
2. Delete sections that do not apply (keep the file short — agents follow short files better).
3. Optionally add a one-line `CLAUDE.md` with `@AGENTS.md` so Claude Code shares this file.

---

### Disclaimer

Written with AI assistance by [AgentKit Co](https://agentkitco.com/). Starting point, not a guarantee — review against your project and keep real enforcement (CI builds, tests, protected prod runs) in place.

### Want more?

- Free templates + generator: https://agentkitco.com/tools/agents-md-generator/
- [AI Coding Agent Config Kit](https://agentkitco.com/products/agent-config-kit/) ($14) — 12-stack AGENTS.md set, Cursor rules, Copilot files
- [AI PR Review Kit](https://agentkitco.com/products/ai-pr-review-kit/) ($17) — blockers-first checklists, paste-ready PR comments, review prompts for Claude Code & Cursor

License: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — use and adapt with attribution to AgentKit Co.
