# AGENTS.md — Godot 4 game (GDScript)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** a Godot 4.x project written in GDScript with static typing, scenes and resources saved as text (`.tscn` / `.tres`), tested with [GUT / gdUnit4], exported to [Windows, Linux, macOS, Web]. Replace bracketed defaults if your setup differs (C# projects need their own build and test commands), and check the docs for your exact Godot version, since editor flags and APIs change between minor releases.

## Project

- Game: [NAME] — [one line on the genre and core loop]
- Godot version: [4.x.y] (pinned; everyone and CI use the same editor build)
- Main scene: [res://scenes/main.tscn] · target platforms: [Windows, Linux, Web]
- Input actions live in Project Settings → Input Map: [move_left, move_right, jump, interact, pause]
- Autoloads (singletons): [GameState, Events, Audio] — keep the list short

## Commands

Use the pinned editor binary as `godot` (adjust the path or name for your machine).

| Task | Command |
|------|---------|
| Import assets headless (first run / CI) | `[godot --headless --import]` |
| Run the game | `[godot --path . ]` or press F5 in the editor |
| Parse-check one script | `[godot --headless --check-only --script res://path/to/file.gd]` |
| Unit tests (GUT) | `[godot --headless -s addons/gut/gut_cmdln.gd -gdir=res://test -gexit]` |
| Format / lint (gdtoolkit) | `[gdformat --check .]` · `[gdlint .]` |
| Export a build | `[godot --headless --export-release "Windows Desktop" build/game.exe]` — **ask first** |
| Publish to Steam / itch.io / a store | **never run by the agent** — a human uploads and releases |

A change is not done until scripts parse with no errors, tests and lint pass, and the main scene runs with no new errors or warnings in the Output / Debugger panel.

## Layout

- `project.godot` — project settings, input map, autoloads (text file; review its diff carefully)
- `scenes/` — `.tscn` scenes grouped by feature (`player/`, `enemies/`, `ui/`, `levels/`)
- `scripts/` or next to each scene — `.gd` files, one script per scene root where possible
- `resources/` — `.tres` data (stats, items, dialogue) built on custom `Resource` classes
- `assets/` — art, audio, fonts (source files like `.blend` / `.psd` in [art-src/] or outside the repo)
- `test/` — GUT / gdUnit4 tests
- `addons/` — third-party plugins, committed as vendored code; don't edit them in place
- Commit `.import` and `.uid` files next to assets; never commit the `.godot/` cache folder, export builds, or `export_credentials.cfg`.

## Conventions

1. **Static typing everywhere.** Type variables, parameters and return values (`var speed: float = 200.0`, `func take_damage(amount: int) -> void:`). Use `class_name` for reusable types. Treat new "untyped declaration" warnings as errors if the project enables them.
2. **Scenes are components.** A scene owns its children; it talks up with signals and down with method calls. No `get_parent().get_parent()` chains or absolute `/root/...` paths. Use `@onready var x: Type = $Child` or `%UniqueName` for nodes you own.
3. **Signals over polling.** Declare typed signals (`signal health_changed(new_value: int)`) and connect with `health_changed.connect(_on_health_changed)`. Use a small `Events` autoload only for truly global events; don't make every system a singleton.
4. **Frame-rate independent.** Movement and physics go in `_physics_process(delta)` and use `delta`; visuals and input polling can use `_process`. Use `move_and_slide()` on `CharacterBody2D/3D` rather than setting positions by hand.
5. **Input through actions.** Read `Input.is_action_pressed("jump")` / `_unhandled_input(event)` with named actions from the Input Map, never hard-coded keys or buttons, so remapping and controllers keep working.
6. **Data in Resources.** Tunables (speeds, damage, drop tables) go in `@export` properties or custom `Resource` files, not magic numbers buried in scripts. Designers should be able to tweak them in the Inspector.
7. **Editing scene files.** `.tscn` / `.tres` are text, but node paths, `ExtResource` ids and `uid://` references must stay consistent. Prefer small, targeted edits; never hand-renumber ids or delete `.uid` files. If a scene change is large, describe it and let a human make it in the editor.
8. **Free what you spawn.** Use `queue_free()` for nodes you no longer need, disconnect signals you connected to long-lived objects, and pool frequently spawned objects (bullets, particles) if profiling shows churn.
9. **Saves are user data.** Save under `user://`, version the save format, and migrate old saves instead of breaking them. Never load untrusted save or mod files with code execution (`load()` of arbitrary `.gd` / `.tscn` from `user://`).
10. **Performance with evidence.** Check the Profiler and Monitors before "optimizing". Mention any change that adds per-frame work in `_process` across many nodes.

## Testing

- Unit-test pure logic (damage math, inventory, save migration, state machines) with [GUT / gdUnit4]; keep it out of nodes where possible so it is easy to test.
- Scene tests: instance the scene in a test, simulate input actions, and assert on signals and state; free it at the end.
- For gameplay feel changes (speed, jump, camera), describe how to verify by hand in the PR — tests can't judge "feels right".

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by refactors, renames or plugin upgrades
- [ ] Scripts are statically typed and parse with no new warnings
- [ ] `project.godot` changes (input map, autoloads, rendering, physics layers) are intentional and listed in the PR
- [ ] Scene/resource diffs touch only the nodes the task needs; `uid://` and `ExtResource` references still resolve
- [ ] Movement uses `delta` / physics process; input uses named actions
- [ ] Old saves still load, or a migration is included
- [ ] Tests and lint pass; the main scene runs with no new errors
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify)

## Boundaries — ask first

- Upgrading the Godot version (even minor) or converting the project between versions
- Exporting release builds, changing export presets, signing, or anything touching store pages and achievements
- Adding, removing or upgrading addons, or switching rendering method (Forward+ / Mobile / Compatibility)
- Renaming or moving many scenes/resources at once (do it in the editor so references update)
- Changing collision layers/masks, autoload order or the save format
- Adding analytics, online features, or new third-party assets with unclear licenses

## How to customize

1. Fill `[BRACKETED]` values and delete options you don't use (test framework, platforms, gdtoolkit).
2. Delete sections that do not apply (keep the file short — agents follow short files better).
3. Optionally add a one-line `CLAUDE.md` with `@AGENTS.md` so Claude Code shares this file.

---

### Disclaimer

Written with AI assistance by [AgentKit Co](https://agentkitco.com/). Starting point, not a guarantee — review against your project, your Godot version's docs and your own playtesting.

### Want more?

- Free templates + generator: https://agentkitco.com/tools/agents-md-generator/
- [AI Coding Agent Config Kit](https://agentkitco.com/products/agent-config-kit/) ($14) — 12-stack AGENTS.md set, Cursor rules, Copilot files
- [AI PR Review Kit](https://agentkitco.com/products/ai-pr-review-kit/) ($17) — blockers-first checklists, paste-ready PR comments, review prompts for Claude Code & Cursor

License: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — use and adapt with attribution to AgentKit Co.
