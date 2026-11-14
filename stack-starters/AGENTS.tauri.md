# AGENTS.md — Tauri 2 desktop app (Rust + TypeScript web frontend)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** a [Tauri 2] desktop app with a Rust core in `src-tauri/` and a [Vite + React / Svelte / Vue] TypeScript frontend in `src/`, using the [npm / pnpm] Tauri CLI. Replace bracketed defaults if your setup differs, and check the current Tauri 2 docs, since plugin names and config keys changed between Tauri 1 and 2.

## Project

- App: [NAME] — [one line on what it does]
- Targets: [Windows / macOS / Linux] (mobile targets only if listed here)
- Frontend: [framework] in `src/`, built to `[dist]` (`build.frontendDist` in `tauri.conf.json`)
- Rust core: `src-tauri/` — commands, state, plugins, OS access
- Plugins in use: [fs, dialog, shell, store, http, updater …] — each one must also be granted in a capability file

## Commands

| Task | Command |
|------|---------|
| Install JS deps | `[npm install]` |
| Dev (frontend + Rust, hot reload) | `[npm run tauri dev]` |
| Frontend type check / lint | `[npm run typecheck]` / `[npm run lint]` |
| Frontend tests | `[npm test]` (Vitest) |
| Rust format check | `cargo fmt --check` (run in `src-tauri/`) |
| Rust lint | `cargo clippy --all-targets -- -D warnings` (in `src-tauri/`) |
| Rust tests | `cargo test` (in `src-tauri/`) |
| Production bundle, signing, notarizing, publishing updates | **never run by the agent** — a human or CI does this |

A change is not done until both the TypeScript and Rust checks pass and the affected flow works in `tauri dev`.

## Layout

- `src/` — frontend UI; talks to Rust only through `invoke` wrappers in `[src/lib/api.ts]`
- `src-tauri/src/lib.rs` — builder setup, plugin registration, `invoke_handler` list
- `src-tauri/src/commands/` — `#[tauri::command]` functions grouped by feature
- `src-tauri/capabilities/*.json` — which windows get which permissions
- `src-tauri/tauri.conf.json` — app identifier, windows, CSP, bundle settings
- `src-tauri/Cargo.toml` / `package.json` — keep Tauri crate and `@tauri-apps/*` package versions on the same major/minor

## Security rules (the important part)

- **Least-privilege capabilities.** Grant only the permissions a feature needs, to only the windows that need them, with path scopes as narrow as possible (e.g. `$APPDATA/**`, not `**`). Never add `*:default` sets or broad `fs`/`shell` scopes "to make it work" — say which permission you added and why.
- **Treat the webview as untrusted.** Every command validates its arguments in Rust (paths, sizes, enums, URLs). Never pass frontend strings straight to `std::process::Command`, the shell plugin, SQL, or file paths without checks; reject `..` traversal and paths outside allowed roots.
- **Keep the CSP.** Don't set `app.security.csp` to `null` or add `unsafe-eval`; don't load remote URLs into a window that has IPC access.
- **No secrets in the frontend bundle.** API keys and tokens live in the Rust side, the OS keychain ([keyring crate / stronghold plugin]), or a backend — never in `VITE_` vars or `localStorage`.
- Open external links with the opener plugin, never by navigating the main window.

## Rust conventions

- Commands return `Result<T, AppError>` where `AppError` implements `serde::Serialize`; no `unwrap()`/`expect()` on anything that can fail at runtime.
- Use `async` commands for disk, network or long work so the UI never freezes; heavy CPU work goes to `tauri::async_runtime::spawn_blocking`.
- Shared state goes through `app.manage(...)` and `State<'_, Mutex<T>>` (prefer `tokio::sync::Mutex` across `.await`); don't hold a lock across an await with `std::sync::Mutex`.
- Push progress to the UI with events (`app.emit` / channels), not polling.
- Every new command is registered in `generate_handler![...]` and has a typed TS wrapper.

## Frontend conventions

- Strict TypeScript; argument names in `invoke('cmd', { … })` match the Rust parameter names (camelCase on the JS side).
- Handle rejected `invoke` promises — show an error, don't swallow it.
- Unlisten event handlers on unmount.
- The app must still render (with a clear message) if a command fails.

## Testing

- Unit-test command logic as plain Rust functions (keep Tauri types at the edges so logic is testable without a window).
- Mock `invoke` in frontend tests (`@tauri-apps/api/mocks`).
- For anything touching files, dialogs or OS integration, describe the manual check you ran in `tauri dev` on [primary OS].

## Review habits

- Small diffs; one feature per change. Explain every capability, permission, scope or CSP change in the PR description.
- Call out new plugins, new crates and new npm packages — each one is attack surface.
- Don't reformat unrelated files or bump unrelated dependencies.

## Ask first (do not do without a human)

- Changing the app `identifier`, bundle settings, signing config, or updater endpoints/public key
- Adding plugins, widening capabilities or scopes, or relaxing the CSP
- Running production builds, signing, notarizing, or publishing releases/updates
- Migrations of on-disk user data or settings formats
- Upgrading Tauri major/minor versions

## How to customize

1. Fill `[BRACKETED]` values and delete plugins and targets you don't use.
2. Delete sections that don't apply (keep the file short — agents follow short files better).
3. Optionally add a one-line `CLAUDE.md` with `@AGENTS.md` so Claude Code shares this file.

---

### Disclaimer

Written with AI assistance by [AgentKit Co](https://agentkitco.com/). Starting point, not a guarantee — review against your project, the current Tauri 2 documentation and security guidance, and your own testing. Tauri is a trademark of its respective owners; this file is not affiliated with or endorsed by the Tauri project.

### Want more?

- Free templates + generator: https://agentkitco.com/tools/agents-md-generator/
- [AI Coding Agent Config Kit](https://agentkitco.com/products/agent-config-kit/) ($14) — 12-stack AGENTS.md set, Cursor rules, Copilot files
- [AI PR Review Kit](https://agentkitco.com/products/ai-pr-review-kit/) ($17) — blockers-first checklists, paste-ready PR comments, review prompts for Claude Code & Cursor

License: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — use and adapt with attribution to AgentKit Co.
