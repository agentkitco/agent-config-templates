# AGENTS.md — Bevy game (Rust)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** a [2D / 3D] game on the Bevy engine [0.x, pinned in `Cargo.toml`], written in stable Rust, organized as one plugin per feature, using Bevy's ECS (components, resources, systems, states), assets loaded from `assets/`, tested with plain `cargo test` plus headless `App` tests, built for [Windows, macOS, Linux, WebAssembly]. Replace bracketed defaults if your setup differs. **Bevy ships breaking changes in most releases** (type names, method names and signatures move), so always check the docs and migration guide for the exact version in `Cargo.toml` before writing code from memory.

## Project

- Game: [NAME] — [one line on the genre and core loop]
- Bevy version: [0.x] (pinned; upgrades follow the official migration guide, one release at a time)
- Rust toolchain: [stable, pinned in `rust-toolchain.toml`] · edition [2021 / 2024]
- Extra crates: [physics e.g. avian / rapier, input e.g. leafwing-input-manager, UI, audio] — each must support our Bevy version
- Entry point: `src/main.rs` builds the `App`; game states: [Loading, MainMenu, InGame, Paused]

## Commands

| Task | Command |
|------|---------|
| Run (dev) | `[cargo run]` (optionally `--features bevy/dynamic_linking` for faster incremental builds on desktop) |
| Check | `cargo check --all-targets` |
| Lint | `cargo clippy --all-targets -- -D warnings` |
| Format | `cargo fmt --all -- --check` |
| Tests | `cargo test` |
| Web build (if used) | `[trunk build / bevy CLI / wasm-bindgen workflow]` |
| Release build | `[cargo build --release]` — **ask first** |
| Publish to Steam / itch.io / a store | **never run by the agent** — a human uploads and releases |

A change is not done until check, clippy, fmt and tests pass and the game runs to the affected state with no new panics or warning logs.

## Layout

- `src/main.rs` — builds the `App`, adds `DefaultPlugins` and our plugins; keep it short
- `src/<feature>/mod.rs` — one `Plugin` per feature (`player`, `enemies`, `ui`, `audio`, `levels`) that registers its own systems, resources, events and states
- `src/components.rs` / per-feature `components.rs` — small data-only components
- `assets/` — textures, audio, fonts, scenes, data files (paths are relative to this folder)
- `tests/` — headless `App` integration tests
- `Cargo.toml` + `Cargo.lock` — commit the lockfile; review every dependency diff
- Never commit `target/`, release builds, signing keys or store credentials

## Conventions

1. **Plugins per feature.** Each feature exposes one `Plugin` that adds its own systems and resources. `main.rs` only composes plugins; don't pile systems into `main`.
2. **Components are data, systems are behavior.** Keep components small and focused (`Health`, `Velocity`, marker structs like `Player`). No game logic in component methods that reach into the world.
3. **Narrow queries.** Query only what a system needs, use filters (`With<T>`, `Without<T>`, `Changed<T>`, `Added<T>`) and split conflicting mutable access with `Without` or `ParamSet` instead of fighting the borrow checker. Prefer change detection over polling every frame.
4. **No panicking lookups.** Don't `unwrap()` single-entity queries or `get()` results in systems; handle the missing case (early return, `let ... else`) since entities can despawn mid-game. Use the fallible APIs your Bevy version provides.
5. **Explicit system order.** When order matters, use `.chain()`, `.before()` / `.after()` or `SystemSet`s; never rely on registration order. Run gameplay systems only in the right state with `run_if(in_state(...))`, and do setup/teardown in `OnEnter` / `OnExit`.
6. **Frame-rate independent.** Scale movement and timers by `Res<Time>` delta; put deterministic physics or simulation in `FixedUpdate`.
7. **Commands are deferred.** Spawns, despawns and inserts through `Commands` apply later, not immediately; don't expect them to be visible in the same system. Despawn whole hierarchies the way your Bevy version recommends so children aren't orphaned.
8. **Events/messages for decoupling.** Systems in different plugins talk through Bevy's event/message types (names differ across versions), not shared mutable globals. Resources are for genuinely global state (score, settings), not a dumping ground.
9. **Assets through `AssetServer`.** Load with `asset_server.load("path")`, keep the `Handle` in a component or resource, and wait for a loading state before use. Don't read files with `std::fs` at runtime — it breaks on web builds.
10. **Saves are user data.** Version the save format, migrate old saves, and treat save and mod files as untrusted input (serde with validation, no panics on bad data).

## Testing

- Pure logic (damage, inventory, level generation) in plain Rust functions with unit tests.
- System tests: build an `App` with `MinimalPlugins` (or none), add the system and resources, spawn entities into the world, call `app.update()`, then assert on components and resources. No window or GPU needed.
- For feel changes (movement, camera, timing), describe how to check by hand in the PR — tests can't judge "feels right".

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by refactors, renames or dependency upgrades
- [ ] Code matches the pinned Bevy version's API (no snippets from older or newer releases)
- [ ] New systems are in the right plugin, schedule and state, with explicit ordering where it matters
- [ ] No new `unwrap()` / `expect()` on queries or lookups inside systems
- [ ] No per-frame allocations or broad queries added to hot systems without a reason
- [ ] Old saves still load, or a migration is included
- [ ] `cargo check`, `clippy -D warnings`, `fmt --check` and `cargo test` pass
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify)

## Boundaries — ask first

- Upgrading Bevy, the Rust toolchain or any Bevy ecosystem crate (they move in lockstep)
- Adding or removing dependencies (and anything with an unclear license or unknown maintainer)
- Changing Cargo features, build profiles, the web build pipeline or target platforms
- Changing the save format, state machine, or global resources other plugins rely on
- Large renames or moves across many files and asset paths
- Release builds, uploads, store pages, achievements, analytics or ads SDKs

## How to customize

1. Fill `[BRACKETED]` values and delete options you don't use (web build, physics crate, dynamic linking).
2. Delete sections that don't apply (keep the file short — agents follow short files better).
3. Optionally add a one-line `CLAUDE.md` with `@AGENTS.md` so Claude Code shares this file.

---

### Disclaimer

Written with AI assistance by [AgentKit Co](https://agentkitco.com/). Starting point, not a guarantee — review against your project, your Bevy version's docs and migration guide, and your own playtesting.

### Want more?

- Free templates + generator: https://agentkitco.com/tools/agents-md-generator/
- [AI Coding Agent Config Kit](https://agentkitco.com/products/agent-config-kit/) ($14) — 12-stack AGENTS.md set, Cursor rules, Copilot files
- [AI PR Review Kit](https://agentkitco.com/products/ai-pr-review-kit/) ($17) — blockers-first checklists, paste-ready PR comments, review prompts for Claude Code & Cursor

License: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — use and adapt with attribution to AgentKit Co.
