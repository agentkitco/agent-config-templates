# AGENTS.md — Flutter (Dart)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** Flutter [3.x stable] with Dart [3.x] and sound null safety, [pub / FVM-pinned SDK], state management with [Riverpod / Bloc / Provider / what exists], routing with [go_router / Navigator 2], tests with `flutter_test` (unit, widget) and [integration_test], linting with [flutter_lints / very_good_analysis]. Replace bracketed defaults if your project differs, and check the current Flutter docs for your SDK version, since APIs and tooling change between releases.

## Project

- App: [PROJECT NAME] — [one-line purpose]
- Platforms: [iOS / Android / web / macOS / Windows / Linux]
- Flavors/environments: [dev / staging / prod via `--flavor` and `--dart-define`]
- Backend/API: [base URL or service] · auth: [provider / none]
- Code generation: [freezed / json_serializable / riverpod_generator / none]

## Commands (repo root)

| Task | Command |
|------|---------|
| Install deps | `[flutter pub get]` |
| Add a package | `[flutter pub add package_name]` (ask first) |
| Run | `[flutter run --flavor dev]` |
| Format | `[dart format .]` |
| Analyze (lint + types) | `[flutter analyze]` |
| Tests | `[flutter test]` |
| Integration tests | `[flutter test integration_test]` |
| Code generation | `[dart run build_runner build --delete-conflicting-outputs]` |
| Release build | `[flutter build appbundle]` / `[flutter build ipa]` (ask first) |

A change is not done until `dart format`, `flutter analyze` (no new warnings) and `flutter test` pass, and generated files are regenerated if their sources changed.

## Layout

- `lib/main.dart` (or `main_<flavor>.dart`) — entry point; keep it thin
- `lib/` — [feature-first `features/<name>/{data,domain,presentation}` / layer-first]; follow what exists
- `test/` mirrors `lib/` · `integration_test/` for on-device flows
- `assets/` and the `flutter:` section of `pubspec.yaml` — images, fonts, translations (`l10n/` with `.arb` files if used)
- `android/`, `ios/` and other platform folders — native config; edit only when the task needs it
- Do not hand-edit generated files (`*.g.dart`, `*.freezed.dart`, `lib/l10n/` output); change the source and rerun generation. Do not touch `build/` or `.dart_tool/`.

## Conventions

1. **Follow the existing structure.** Match current folders, naming and patterns. Don't add a second state manager, router or DI approach the repo doesn't already use.
2. **Small widgets, `const` where possible.** Split big `build` methods into widget classes (not helper methods returning widgets), and mark constructors and instances `const` when you can.
3. **No side effects in `build`.** Don't start network calls, timers or navigation from `build`; use the state manager, `initState`, or the repo's pattern.
4. **Dispose what you create.** Controllers, streams, subscriptions, focus nodes and animation controllers get disposed or cancelled.
5. **`BuildContext` across async gaps.** After an `await`, check `mounted` (or `context.mounted`) before using `context`.
6. **Null safety honestly.** Avoid `!` unless the value truly can't be null; never silence the analyzer with `// ignore:` without a reason in the PR.
7. **Data in repositories/services.** Widgets talk to state/providers, which talk to repositories; no raw `http` calls in widgets. Handle loading, empty, error and offline states.
8. **Immutable models.** Use `final` fields and `copyWith` (or freezed) for state; parse JSON in one place with typed models.
9. **Lists.** Use `ListView.builder`/`SliverList` for long or dynamic lists with stable `Key`s where items reorder.
10. **Theme and l10n.** Use `Theme.of(context)` tokens and the repo's localization, not hard-coded colors, sizes or user-facing strings.
11. **Secrets.** Anything compiled into the app (`--dart-define`, assets, source) can be extracted, so treat it as public. Keep real secrets on a server, and store tokens with [flutter_secure_storage], not `shared_preferences`.
12. **Platform channels and permissions.** Ask before adding a plugin with native code, a platform channel, or a new permission entry in `AndroidManifest.xml` or `Info.plist`.
13. **Accessible by default.** Add `Semantics`/`tooltip` labels for icon-only controls, keep tap targets at least 48x48, and check with larger text scale.

## Testing

- Unit-test pure logic and repositories; widget-test screens with `pumpWidget`, finding by text, key or semantics label, and driving them with `tester.tap` and `pumpAndSettle`.
- Override providers or inject fakes at the boundary instead of hitting the network.
- New behavior gets a test; a bug fix gets a test that fails before the fix where practical. Golden tests only if the repo already uses them.

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by refactors or reformatting of untouched files
- [ ] `dart format`, `flutter analyze` and `flutter test` pass; generated files are up to date
- [ ] Checked on [Android and iOS] (emulator, simulator or device) and said which in the PR
- [ ] No new `!`, `dynamic` or `// ignore:` without reason; no secrets in source, assets or `--dart-define`
- [ ] Controllers and subscriptions disposed; no `context` use after `await` without a `mounted` check
- [ ] Loading, empty, error and offline states handled; long lists built lazily
- [ ] New UI has semantics labels and works with larger text
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify)

## Boundaries — ask first

- Adding, removing or upgrading a package (especially one with native code), or bumping the Flutter/Dart SDK or `environment:` constraint
- Editing `android/`, `ios/` or other platform folders, signing configs, bundle IDs, flavors or permissions
- Building release artifacts or uploading anything to the Play Console, App Store Connect or Firebase

## How to customize

1. Fill `[BRACKETED]` values and delete options you don't use (platforms, flavors, code generation).
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
