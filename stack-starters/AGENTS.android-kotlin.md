# AGENTS.md — Android (Kotlin + Jetpack Compose)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** Android app in Kotlin [2.x] with Jetpack Compose and Material 3, Gradle with the Kotlin DSL and a version catalog (`gradle/libs.versions.toml`), [Hilt / Koin / manual] DI, ViewModel + `StateFlow` for UI state, coroutines, [Retrofit / Ktor] for networking, [Room / DataStore] for local data, tests with JUnit, [Turbine / MockK / fakes] and Compose UI tests. Replace bracketed defaults if your project differs, and check the current Android and Kotlin docs for your versions, since APIs, Gradle plugins and lint rules change between releases.

## Project

- App: [PROJECT NAME] — [one-line purpose]
- `minSdk` [24] · `targetSdk` [latest required by Play] · application ID: [com.example.app]
- Modules: [`:app` only / `:app`, `:core:*`, `:feature:*`]
- Build variants: [debug / release] · flavors: [none / dev, prod]
- Backend/API: [base URL or service] · auth: [provider / none]

## Commands (repo root)

| Task | Command |
|------|---------|
| Build debug | `[./gradlew assembleDebug]` |
| Install on device/emulator | `[./gradlew installDebug]` |
| Unit tests | `[./gradlew testDebugUnitTest]` |
| Instrumented / Compose UI tests | `[./gradlew connectedDebugAndroidTest]` (needs a running emulator) |
| Lint | `[./gradlew lintDebug]` |
| Format / static analysis | `[./gradlew ktlintCheck]` / `[./gradlew detekt]` / `[./gradlew spotlessCheck]` (whichever exists) |
| Release build | `[./gradlew bundleRelease]` (ask first) |

A change is not done until it builds, unit tests pass, and lint plus the repo's formatter show no new issues. Run instrumented tests when you touched UI or anything under `androidTest/`, and say in the PR if you could not.

## Layout

- `app/src/main/java/` (or `kotlin/`) — [feature-first `feature/<name>/{ui,data,domain}` / layer-first]; follow what exists
- `app/src/main/res/` — strings, drawables, themes; `AndroidManifest.xml` — components and permissions
- `app/src/test/` — JVM unit tests · `app/src/androidTest/` — instrumented and Compose UI tests
- `gradle/libs.versions.toml` — every dependency version lives here, not inline in `build.gradle.kts`
- Do not edit `build/`, `.gradle/` or generated code (Room, Hilt, KSP output); change the source and rebuild.

## Conventions

1. **Follow the existing structure.** Match current modules, packages and patterns. Don't add a second DI framework, navigation library or networking client.
2. **Unidirectional data flow.** ViewModels expose immutable `StateFlow<UiState>`; screens send events up and render state down. Don't expose `MutableStateFlow` or mutable lists from a ViewModel.
3. **Collect lifecycle-aware.** In Compose use `collectAsStateWithLifecycle()`; in views use `repeatOnLifecycle`. Never `GlobalScope`; use `viewModelScope` or `lifecycleScope`.
4. **Stateless composables.** Hoist state; pass values and lambdas in. Keep side effects in `LaunchedEffect`/`DisposableEffect` with correct keys, and never start work directly in a composable body.
5. **Previews.** Add a `@Preview` for new screens and components using fake state, not a real ViewModel.
6. **Main thread stays free.** Disk, network and database work runs on injected dispatchers (`Dispatchers.IO` behind an interface), not hard-coded in classes you test.
7. **Data in repositories.** UI talks to ViewModels, ViewModels to repositories/use cases; no Retrofit or DAO calls in composables. Handle loading, empty, error and offline states.
8. **Null safety honestly.** Avoid `!!` unless the value truly can't be null; don't suppress lint or warnings without a reason in the PR.
9. **Resources, not literals.** User-facing text goes in `strings.xml`; colors and typography come from `MaterialTheme`, not hard-coded values.
10. **Lists.** Use `LazyColumn`/`LazyRow` with stable `key`s for dynamic items.
11. **Configuration changes and process death.** State that must survive goes in the ViewModel or `SavedStateHandle`/`rememberSaveable`, not plain `remember`.
12. **Secrets.** Anything in the APK (`BuildConfig`, resources, source) can be extracted, so treat it as public. Keep real secrets on a server; store tokens with encrypted or Keystore-backed storage, not plain `SharedPreferences`. Never commit keystores or `local.properties`.
13. **Accessible by default.** Give icon-only controls a `contentDescription`, keep touch targets at least 48dp, and check with large font scale and TalkBack.

## Testing

- Unit-test ViewModels and repositories with fakes; use `runTest` and a test dispatcher, and Turbine (or `first()`/`toList()`) for flows.
- Compose UI tests use `createComposeRule()`, find nodes by text, test tag or content description, and assert state, not pixels.
- New behavior gets a test; a bug fix gets a test that fails before the fix where practical. Screenshot tests only if the repo already uses them.

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by refactors or reformatting of untouched files
- [ ] Builds; unit tests, lint and formatter pass; instrumented tests run or noted as not run
- [ ] Checked on [emulator API level / device] and said which in the PR
- [ ] No new `!!`, `@Suppress` or `GlobalScope`; no secrets, keystores or `local.properties` committed
- [ ] Flows collected lifecycle-aware; state survives rotation where it should
- [ ] Loading, empty, error and offline states handled; lazy lists have stable keys
- [ ] New UI has content descriptions, uses string resources and theme tokens, and works with large fonts
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify)

## Boundaries — ask first

- Adding, removing or upgrading dependencies or Gradle plugins, or changing AGP, Kotlin, `compileSdk`, `minSdk` or `targetSdk`
- Adding permissions, exported components, deep links or anything else in `AndroidManifest.xml`
- Changing signing configs, application ID, ProGuard/R8 rules, or database schema/migrations
- Building release artifacts or uploading anything to the Play Console or Firebase

## How to customize

1. Fill `[BRACKETED]` values and delete options you don't use (modules, flavors, DI, formatter).
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
