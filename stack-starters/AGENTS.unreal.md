# AGENTS.md — Unreal Engine 5 game (C++ + Blueprints)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** a game on Unreal Engine [5.x, set in the `.uproject` `EngineAssociation`], gameplay code in C++ modules under `Source/`, designers working in Blueprints and the editor, Enhanced Input for controls, [Gameplay Ability System / plain components] for abilities, assets in `Content/` tracked with [Git LFS / Perforce], targeting [Windows, consoles, mobile]. Replace bracketed defaults if your setup differs. **Agents cannot open or diff binary `.uasset` / `.umap` files**, so any change that needs the editor is described for a human instead of faked.

## Project

- Game: [NAME] — [one line on the genre and core loop]
- Engine: Unreal Engine [5.x] ([launcher / source build]); upgrades are a human decision
- Primary module: `Source/[ProjectName]/`, API macro `[PROJECTNAME]_API`
- Extra modules / plugins: [list game feature plugins, marketplace or Fab plugins]
- Version control: [Git + LFS with file locking / Perforce] — binary assets must be locked before editing

## Commands

| Task | Command |
|------|---------|
| Generate project files | `[GenerateProjectFiles / UnrealBuildTool -projectfiles]` |
| Build editor target | `[Build.bat <Project>Editor Win64 Development -Project="<path>.uproject"]` (Linux/Mac: `Build.sh`) |
| Run automation tests (headless) | `[UnrealEditor-Cmd "<path>.uproject" -ExecCmds="Automation RunTests <Project>; Quit" -unattended -nullrhi -nopause]` |
| Lint / format C++ | `[clang-format -i on changed files]` · [static analysis if configured] |
| Package / cook a build | `[RunUAT BuildCookRun ...]` — **ask first** |
| Upload to Steam / Epic / console portals | **never run by the agent** — a human packages, uploads and releases |

A change is not done until the editor target compiles with no new warnings, automation tests pass, and the change is checked in PIE (Play In Editor) — say so explicitly if you could not run the editor and a human must check it.

## Layout

- `Source/[ProjectName]/` — game module: `Public/` headers, `Private/` implementation, `[ProjectName].Build.cs` for module dependencies
- `Source/[ProjectName]/Tests/` — automation tests (or a separate test module)
- `Plugins/` — game feature and third-party plugins (each with its own `.uplugin`)
- `Config/` — `DefaultEngine.ini`, `DefaultGame.ini`, `DefaultInput.ini`, etc. (text, reviewable)
- `Content/` — binary assets: Blueprints, maps, materials, meshes, audio (editor only)
- Never commit `Binaries/`, `Intermediate/`, `Saved/`, `DerivedDataCache/` or `.vs/`; never commit signing keys, platform SDK credentials or store tokens

## Conventions

1. **C++ for systems, Blueprints for content.** Core logic, performance-sensitive code and anything that needs tests lives in C++; expose tuning and hooks with `UPROPERTY(EditAnywhere/EditDefaultsOnly, BlueprintReadOnly, Category="...")` and `UFUNCTION(BlueprintCallable / BlueprintImplementableEvent)` so designers extend it without editing C++.
2. **Follow Epic's naming.** Prefixes `A` (actors), `U` (UObjects/components), `F` (structs), `E` (enums), `I` (interfaces), `b` for bools; one class per header/source pair; `#pragma once`, include what you use and keep `.generated.h` as the last include.
3. **Reflection keeps objects alive.** Every `UObject*` member is a `UPROPERTY()` (or `TObjectPtr<>`) so garbage collection sees it; use `TWeakObjectPtr` for non-owning references and never hold raw `UObject*` across frames without one. Create objects with `NewObject` / `CreateDefaultSubobject` / `SpawnActor`, never `new`.
4. **Null-safe, no crashes.** Check pointers and use `IsValid()` before use; use `ensure`/`ensureMsgf` for "should never happen" in shipping code and keep `check` for true invariants. No `GetAllActorsOfClass` or `FindComponentByClass` in `Tick`.
5. **Tick sparingly.** Disable `PrimaryActorTick.bCanEverTick` unless needed; prefer timers, delegates and events. Scale anything time-based by `DeltaTime`.
6. **Respect the network model.** [If multiplayer:] the server is authoritative; replicate with `UPROPERTY(Replicated / ReplicatedUsing=OnRep_X)` and `GetLifetimeReplicatedProps`, validate every Server RPC's inputs, and never trust a client value for damage, currency or inventory.
7. **Don't break Blueprints and saved assets.** Renaming or removing a `UPROPERTY`, `UFUNCTION`, class or enum value can break Blueprints, maps and save games silently. Add a redirect in `DefaultEngine.ini` (`+PropertyRedirects` / `+ClassRedirects` / `+FunctionRedirects`) and flag it in the PR.
8. **Module dependencies are deliberate.** Add a module to `.Build.cs` only when needed, keep `Public`/`Private` dependency lists accurate, and never make the runtime module depend on editor-only modules (wrap editor code in `#if WITH_EDITOR` or an editor module).
9. **Enhanced Input and data assets.** Input goes through Input Actions and Mapping Contexts, not hard-coded keys; tunables and item/ability data live in Data Assets or Data Tables, not magic numbers in code.
10. **Saves are user data.** Version `USaveGame` subclasses, handle old versions, and treat loaded and mod data as untrusted.

## Testing

- Pure logic (damage formulas, inventory, procedural generation) in plain C++ functions or structs covered by automation tests (`IMPLEMENT_SIMPLE_AUTOMATION_TEST` or the Automation Spec framework), runnable headless with `-nullrhi`.
- Functional tests or Gauntlet for map-level behavior, if the project has them.
- For feel and visuals (movement, camera, VFX, animation) describe the PIE steps a human should take in the PR — tests cannot judge "feels right".

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by refactors, renames or engine/plugin upgrades
- [ ] Code matches the project's engine version's API (no snippets from UE4 or other 5.x versions)
- [ ] Every `UObject*` member is reflected; no raw `new`, no unchecked pointer use
- [ ] Renamed or removed reflected members have redirects, or none were renamed
- [ ] No new per-tick searches, allocations or always-on ticking without a reason
- [ ] Any Blueprint, map or asset change needed is listed for a human, not claimed as done
- [ ] Editor target builds with no new warnings and automation tests pass
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify in PIE)

## Boundaries — ask first

- Changing the engine version, `EngineAssociation`, or enabling/disabling/updating plugins
- Editing `.uproject`, `.uplugin`, `.Build.cs` / `.Target.cs` beyond a needed module dependency
- Large changes to `Config/*.ini` (rendering, platform, networking, input defaults)
- Changing replication, save game format, or core gameplay classes other systems rely on
- Moving or renaming many classes, folders or assets (redirectors and Blueprints are easy to break)
- Packaging, cooking, signing, store pages, achievements, analytics or ads SDKs

## How to customize

1. Fill `[BRACKETED]` values and delete options you don't use (multiplayer, GAS, consoles).
2. Delete sections that don't apply (keep the file short — agents follow short files better).
3. Optionally add a one-line `CLAUDE.md` with `@AGENTS.md` so Claude Code shares this file.

---

### Disclaimer

Written with AI assistance by [AgentKit Co](https://agentkitco.com/). Starting point, not a guarantee — review against your project, your engine version's documentation, and your own playtesting. Unreal Engine is a trademark of Epic Games; this file is not affiliated with or endorsed by Epic Games.

### Want more?

- Free templates + generator: https://agentkitco.com/tools/agents-md-generator/
- [AI Coding Agent Config Kit](https://agentkitco.com/products/agent-config-kit/) ($14) — 12-stack AGENTS.md set, Cursor rules, Copilot files
- [AI PR Review Kit](https://agentkitco.com/products/ai-pr-review-kit/) ($17) — blockers-first checklists, paste-ready PR comments, review prompts for Claude Code & Cursor

License: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — use and adapt with attribution to AgentKit Co.
