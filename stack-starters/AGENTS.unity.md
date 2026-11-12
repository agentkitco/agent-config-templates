# AGENTS.md — Unity game (C#)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** a Unity [6 / 2022 LTS] project in C#, scripts split into assembly definitions (`.asmdef`), assets serialized as text with visible `.meta` files, tested with the Unity Test Framework (Edit Mode + Play Mode), using [the Input System package], built for [Windows, macOS, WebGL]. Replace bracketed defaults if your setup differs, and check the docs for your exact editor version, since APIs and command-line flags change between releases.

## Project

- Game: [NAME] — [one line on the genre and core loop]
- Unity version: [6000.0.x / 2022.3.x] (pinned in `ProjectSettings/ProjectVersion.txt`; everyone and CI use the same editor build)
- Render pipeline: [URP / HDRP / Built-in] · target platforms: [Windows, macOS, WebGL]
- Bootstrap scene: [Assets/Scenes/Boot.unity] · scenes in the build list: [Boot, MainMenu, Game]
- Input actions asset: [Assets/Input/PlayerControls.inputactions] with maps [Player, UI]

## Commands

Use the pinned editor binary as `Unity` (adjust the path for your machine, e.g. Unity Hub's install folder). Batch mode can't run while the same project is open in the editor.

| Task | Command |
|------|---------|
| Edit Mode tests | `[Unity -batchmode -projectPath . -runTests -testPlatform EditMode -testResults results-edit.xml -logFile -]` |
| Play Mode tests | `[Unity -batchmode -projectPath . -runTests -testPlatform PlayMode -testResults results-play.xml -logFile -]` |
| Compile check (no editor) | open the project or run the Edit Mode tests; a compile error fails both |
| Format (if used) | `[dotnet format whitespace --folder Assets/Scripts --verify-no-changes]` |
| Build a player | `[Unity -batchmode -quit -projectPath . -executeMethod Build.CI.BuildWindows -logFile -]` — **ask first** |
| Publish to Steam / itch.io / a console or mobile store | **never run by the agent** — a human uploads and releases |

A change is not done until the project compiles with no new errors or warnings in the Console, Edit Mode and Play Mode tests pass, and the affected scene enters Play Mode with no new errors.

## Layout

- `Assets/Scripts/` — C# grouped by feature (`Player/`, `Enemies/`, `UI/`, `Core/`), each area with its own `.asmdef`
- `Assets/Tests/EditMode/` and `Assets/Tests/PlayMode/` — test assemblies referencing the code they test
- `Assets/Prefabs/`, `Assets/Scenes/`, `Assets/Data/` (ScriptableObject assets), `Assets/Art/`, `Assets/Audio/`
- `Assets/Plugins/` and imported Asset Store packages — vendored; don't edit in place
- `Packages/manifest.json` + `packages-lock.json` — package versions (review every diff)
- `ProjectSettings/` — commit it; changes here affect every scene and build
- Commit every `.meta` file with its asset. Never commit `Library/`, `Temp/`, `Obj/`, `Logs/`, `UserSettings/`, `Build/` or signing keystores.

## Conventions

1. **`.meta` files are identity.** A `.meta` holds the asset's GUID that scenes and prefabs reference. Move or rename assets in the editor (or move the `.meta` with the file); never delete or regenerate a `.meta`, or references silently break.
2. **Serialized fields, not public fields.** Use `[SerializeField] private float moveSpeed = 5f;` for Inspector values. Renaming a serialized field loses saved values unless you add `[FormerlySerializedAs("oldName")]`.
3. **Cache, don't search.** Get components in `Awake` (or assign them in the Inspector) and cache them. No `GetComponent`, `FindObjectOfType` / `FindFirstObjectByType`, `GameObject.Find` or `Camera.main` lookups inside `Update`.
4. **Frame-rate independent.** Use `Time.deltaTime` in `Update`; put physics (forces, `Rigidbody` velocity) in `FixedUpdate`. Don't move a `Rigidbody` by setting `transform.position`.
5. **Input through actions.** Read input from the Input System actions asset (or a generated wrapper class), never hard-coded `KeyCode`s, so rebinding and gamepads keep working.
6. **Data in ScriptableObjects.** Tunables (stats, waves, loot tables, dialogue) live in ScriptableObject assets, not magic numbers in scripts. Don't write to ScriptableObject assets at runtime as if they were save data — in the editor those changes persist to disk.
7. **Unity null checks.** Destroyed `UnityEngine.Object`s compare equal to `null` but aren't truly null, so `?.` and `??` skip Unity's check. Use `if (thing != null)` or `TryGetComponent` for anything deriving from `UnityEngine.Object`.
8. **Low garbage in hot paths.** Avoid LINQ, string concatenation, boxing and `new` collections in `Update`. Pool frequently spawned objects (`UnityEngine.Pool.ObjectPool<T>`) when the Profiler shows churn. Unsubscribe events in `OnDisable` / `OnDestroy`.
9. **Scenes and prefabs are YAML — edit with care.** Hand-editing `.unity` / `.prefab` / `.asset` files risks broken `fileID` / `guid` references and nasty merges. Prefer code, prefab variants and small focused changes; for structural scene or prefab work, describe the change and let a human make it in the editor.
10. **Saves are user data.** Save under `Application.persistentDataPath`, version the format, migrate old saves instead of breaking them, and don't use `BinaryFormatter` (insecure and obsolete). Treat save and mod files as untrusted input.

## Testing

- Edit Mode tests for pure C# logic (damage math, inventory, state machines, save migration). Keep that logic in plain classes, not MonoBehaviours, so it's easy to test.
- Play Mode tests (`[UnityTest]` coroutines) for behaviour that needs frames, physics or a loaded scene; clean up spawned objects in `[TearDown]`.
- For gameplay feel changes (movement, camera, timing), describe how to check by hand in the PR — tests can't judge "feels right".

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by refactors, renames or package upgrades
- [ ] Every new or moved asset has its `.meta`; no `.meta` deleted or regenerated
- [ ] Renamed serialized fields use `[FormerlySerializedAs]`
- [ ] No per-frame `Find` / `GetComponent` / allocations added to `Update`
- [ ] `ProjectSettings/` and `Packages/manifest.json` changes are intentional and listed in the PR
- [ ] Scene / prefab YAML diffs touch only what the task needs
- [ ] Old saves still load, or a migration is included
- [ ] Edit Mode + Play Mode tests pass; Console shows no new errors or warnings
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify)

## Boundaries — ask first

- Upgrading the Unity editor version or switching render pipeline
- Adding, removing or upgrading packages or Asset Store assets (and anything with an unclear license)
- Changing Player Settings, scripting backend (Mono / IL2CPP), build profiles, signing or the scenes-in-build list
- Moving or renaming many assets at once (do it in the editor so references update)
- Changing physics layers / collision matrix, tags, script execution order or the save format
- Building players, uploading builds, or touching store pages, achievements, analytics or ads SDKs

## How to customize

1. Fill `[BRACKETED]` values and delete options you don't use (render pipeline, platforms, formatter).
2. Delete sections that don't apply (keep the file short — agents follow short files better).
3. Optionally add a one-line `CLAUDE.md` with `@AGENTS.md` so Claude Code shares this file.

---

### Disclaimer

Written with AI assistance by [AgentKit Co](https://agentkitco.com/). Starting point, not a guarantee — review against your project, your Unity version's docs and your own playtesting.

### Want more?

- Free templates + generator: https://agentkitco.com/tools/agents-md-generator/
- [AI Coding Agent Config Kit](https://agentkitco.com/products/agent-config-kit/) ($14) — 12-stack AGENTS.md set, Cursor rules, Copilot files
- [AI PR Review Kit](https://agentkitco.com/products/ai-pr-review-kit/) ($17) — blockers-first checklists, paste-ready PR comments, review prompts for Claude Code & Cursor

License: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — use and adapt with attribution to AgentKit Co.
