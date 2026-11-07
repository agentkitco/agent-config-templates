# AGENTS.md — Rust Axum HTTP service / CLI

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** Rust [1.75+], Cargo (`Cargo.toml`), [workspace or single crate], Axum [0.7+] for HTTP (or clap for CLI). Replace bracketed defaults if your project differs.

## Project

- App: [PROJECT NAME] — [one-line purpose]
- Runtime: Rust [1.75+] · Crate / workspace: `[crate_name]`
- Shape: [Axum HTTP service / CLI / both]

## Commands (repo root)

| Task | Command |
|------|---------|
| Build | `cargo build` · release: `cargo build --release` |
| Run (bin) | `cargo run --bin [app]` · with features: `cargo run --bin [app] --features [feat]` |
| Unit / integration tests | `cargo test` · one test: `cargo test [name]` · one package: `cargo test -p [crate]` |
| Format | `cargo fmt` · check only: `cargo fmt -- --check` |
| Lint | `cargo clippy --all-targets -- -D warnings` |
| Check (no codegen) | `cargo check` |

A change is not done until `cargo test` and `cargo clippy` for touched crates pass. Prefer the narrowest `-p` / test filter while iterating; run the full suite before handing off.

## Layout

- `src/main.rs` or `src/bin/[app].rs` — binary entry (thin; wire dependencies here)
- `src/lib.rs` — library root when the crate is both lib + bin
- `src/[module]/` — handlers, services, domain (prefer modules over a flat god file)
- `tests/` — integration tests (black-box against the public API)
- Workspace (if used): members under `[crates/|services/]`; shared code in a small `common` / `shared` crate

## Conventions

1. **Errors.** Prefer `Result<T, E>` everywhere in library paths. Use `thiserror` for typed public errors and/or `anyhow` at binary / handler boundaries. Do not `.unwrap()` / `.expect()` on fallible library paths — reserve for tests and true invariants with a clear message.
2. **Async.** Axum handlers are async; keep CPU-heavy work off the async runtime (`spawn_blocking` or a dedicated pool). Propagate `CancellationToken` / request-scoped state explicitly when needed.
3. **HTTP (Axum).** Extractors for typed input; validate early; map domain errors to stable status codes via `IntoResponse`. Prefer tower middleware for cross-cutting concerns (timeouts, tracing, auth layers).
4. **Types.** Prefer owned types at API boundaries; borrow inside hot loops. Avoid `clone()` as a default escape hatch — justify it.
5. **Config.** Load from env / clap once at startup; fail fast on missing required values. Never log secrets.
6. **Logging / tracing.** Prefer `tracing` spans over ad-hoc `println!`. No secrets, tokens, or PII in spans or logs.
7. **Dependencies.** Ask before adding a crate. Prefer well-maintained crates already in the tree. Pin via `Cargo.toml` / `Cargo.lock` only through `cargo add` / `cargo update` as the project already does.

## Review habits (before you say “done”)

- [ ] Diff is scoped to the ask — no drive-by refactors
- [ ] No new `.unwrap()` / `.expect()` in library or handler paths without a documented invariant
- [ ] Errors carry context; no swallowed `Result`
- [ ] No secrets, tokens, or PII in logs or commits
- [ ] `cargo fmt -- --check`, `cargo clippy`, and `cargo test` pass on touched crates
- [ ] If you opened a PR: blockers-first summary (what broke if wrong → what changed → how to verify)

## Boundaries — ask first

- Adding a crate dependency, changing the MSRV / edition, CI, auth, or datastore schema
- Deleting public modules, rewriting git history, force-pushing, or touching production infra

## Never

- Commit `target/` or edit `Cargo.lock` by hand on a library-only publish workflow the repo does not already use
- Disable clippy lints broadly to silence warnings (`allow` only with a one-line why)
- Claim a task is done when tests, fmt, or clippy failed
- Use `unsafe` without an explicit `// SAFETY:` justification and review

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
