# AGENTS.md — Roblox experience (Luau + Rojo)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** a Roblox experience written in Luau and synced into Roblox Studio with [Rojo], with tools pinned by [Rokit / Aftman / Foreman], packages from [Wally], linted with [Selene], formatted with [StyLua], and type-checked with [luau-lsp] in `--!strict` mode. Replace bracketed defaults if your setup differs, and check the current docs for each tool, since Roblox APIs and tool flags change often.

## Project

- Experience: [NAME] — [one line on the genre and core loop]
- Rojo project file: [default.project.json] (maps `src/` folders to Roblox services)
- Place for local testing: [build.rbxl] (built from the project file; never edit the published place directly)
- Networking model: **server-authoritative** — the server owns game state, the client only sends requests and renders
- Data: [DataStoreService via ProfileStore / a thin wrapper in `src/server/Data/`]

## Commands

| Task | Command |
|------|---------|
| Install pinned tools | `[rokit install]` |
| Install packages | `[wally install]` |
| Live sync into Studio | `[rojo serve]` (then connect from the Rojo Studio plugin) |
| Build a place file | `[rojo build -o build.rbxl]` |
| Lint | `[selene src]` |
| Format check | `[stylua --check src]` |
| Type check | `[luau-lsp analyze --definitions=globalTypes.d.luau --sourcemap=sourcemap.json src]` (generate the sourcemap with `rojo sourcemap -o sourcemap.json`) |
| Tests | `[run the TestEZ / Jest-Lua runner in Studio or via run-in-roblox / Open Cloud Luau execution]` |
| Publish the place or update a live experience | **never run by the agent** — a human publishes from Studio or CI |

A change is not done until lint, format and type checks pass, tests pass, and the affected flow works in a Studio play test (including a multi-client **Test → Clients and Servers** run for anything networked).

## Layout

- `src/server/` → `ServerScriptService` — game logic, data, anti-cheat checks (never visible to clients)
- `src/client/` → `StarterPlayer.StarterPlayerScripts` — input, camera, UI controllers
- `src/shared/` → `ReplicatedStorage.Shared` — ModuleScripts used by both sides (types, constants, pure logic)
- `src/shared/Remotes` or a single `Remotes` module — every RemoteEvent / RemoteFunction declared in one place
- `Packages/`, `ServerPackages/` — Wally output; don't edit, don't hand-commit changes
- `wally.toml` + `wally.lock`, `rokit.toml` (or `aftman.toml`), `selene.toml`, `stylua.toml` — review every diff
- File names drive script type under Rojo: `*.server.luau` = Script, `*.client.luau` = LocalScript, plain `*.luau` = ModuleScript, `init.luau` makes a folder a module

## Conventions

1. **Never trust the client.** Every RemoteEvent / RemoteFunction handler on the server validates the player, argument types, ranges, cooldowns and ownership before acting. Clients request ("I want to buy item 12"); the server decides and applies.
2. **Server owns state.** Currency, inventory, damage, cooldowns and purchases are computed on the server. Don't let a client set values that replicate into other players' games or into saved data.
3. **No client-invoked yields on the server.** Don't use `RemoteFunction:InvokeClient` — a client can hang or error it. Use RemoteEvents for server → client.
4. **Strict types.** Start new modules with `--!strict`, type function parameters and returns, and export shared types from `src/shared/Types`. Fix type errors; don't silence them with `any` casts.
5. **Modern APIs.** Use `task.wait` / `task.spawn` / `task.defer` / `task.delay`, never the deprecated `wait` / `spawn` / `delay`. Use `:GetAttribute` / `:SetAttribute` for simple per-instance data.
6. **Clean up connections.** Store `RBXScriptConnection`s and disconnect them (or use a Trove / Maid / Janitor) when a character, UI or object goes away. Handle `Players.PlayerRemoving` and `player.CharacterAdded` re-spawns.
7. **Wait for replicated instances on the client.** Use `WaitForChild` (with a timeout where sensible) for things the server creates; don't assume they exist on the first frame.
8. **DataStores are fragile and rate-limited.** Wrap every call in `pcall`, retry with backoff, session-lock (e.g. ProfileStore) to avoid duplication, save on `PlayerRemoving` and `game:BindToClose`, and version the data shape with migrations. Never change a DataStore name or key format without a migration plan.
9. **Monetization goes through `ProcessReceipt`.** Developer product grants happen only inside `MarketplaceService.ProcessReceipt`, are idempotent by `PurchaseId`, and return `PurchaseGranted` only after the grant is saved. Never grant from a client message.
10. **Performance budget.** Avoid per-frame work in `RunService.Heartbeat` / `RenderStepped` unless needed, don't create instances in hot loops, and stream-friendly code must handle parts that aren't loaded yet when `StreamingEnabled` is on.

## Testing

- Pure logic (damage formulas, shop pricing, inventory, data migrations) lives in plain ModuleScripts with no service calls, so it can be tested with [TestEZ / Jest-Lua].
- Use mock DataStores or a Studio test place for data code; never point tests at the live experience's DataStores.
- For gameplay feel and UI, describe the Studio play-test steps (single player and multi-client) a human should run in the PR — tests can't judge "feels right".

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by refactors, renames or package/tool upgrades
- [ ] Every new or changed remote handler validates its arguments and rate-limits on the server
- [ ] No game state, currency or purchase logic moved to the client
- [ ] DataStore calls are wrapped in `pcall`, and any data-shape change has a migration
- [ ] Connections are cleaned up; no deprecated `wait` / `spawn` / `delay`
- [ ] Any Studio-only change (models, terrain, UI layout, place settings) is listed for a human, not claimed as done
- [ ] Lint, format, type check and tests pass; multi-client play test noted
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify in Studio)

## Boundaries — ask first

- Publishing, updating the live place, or changing experience/game settings
- DataStore names, key formats, or anything that could wipe or duplicate player data
- Developer products, game passes, prices, Robux flows or `ProcessReceipt`
- Adding or upgrading Wally packages or pinned tools
- HttpService / external webhooks, third-party analytics or ads, and anything touching chat, text filtering or age-gated features
- Large moves or renames of services/folders other scripts reference by path

## How to customize

1. Fill `[BRACKETED]` values and delete tools you don't use (e.g. no Wally, no tests yet).
2. Delete sections that don't apply (keep the file short — agents follow short files better).
3. Optionally add a one-line `CLAUDE.md` with `@AGENTS.md` so Claude Code shares this file.

---

### Disclaimer

Written with AI assistance by [AgentKit Co](https://agentkitco.com/). Starting point, not a guarantee — review against your project, the current Roblox Creator documentation, Roblox's Community Standards and Terms of Use, and your own play testing. Roblox and Roblox Studio are trademarks of Roblox Corporation; this file is not affiliated with or endorsed by Roblox Corporation.

### Want more?

- Free templates + generator: https://agentkitco.com/tools/agents-md-generator/
- [AI Coding Agent Config Kit](https://agentkitco.com/products/agent-config-kit/) ($14) — 12-stack AGENTS.md set, Cursor rules, Copilot files
- [AI PR Review Kit](https://agentkitco.com/products/ai-pr-review-kit/) ($17) — blockers-first checklists, paste-ready PR comments, review prompts for Claude Code & Cursor

License: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — use and adapt with attribution to AgentKit Co.
