# AGENTS.md — Expo (React Native + TypeScript)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** Node [20+ / 22 LTS], [npm / pnpm / yarn], Expo SDK [5x] with Expo Router (file-based routes in `app/`), React Native with TypeScript in strict mode, tests with [Jest + jest-expo + React Native Testing Library], linting with [eslint-config-expo], builds with EAS. Replace bracketed defaults if your project differs, and check the current Expo docs for your SDK version, since APIs and config change between SDKs.

## Project

- App: [PROJECT NAME] — [one-line purpose]
- Platforms: [iOS / Android / web]
- Workflow: [managed with Continuous Native Generation (no committed `ios/` `android/`) / bare with committed native folders]
- Backend/API: [base URL or service] · auth: [provider / none]
- State/data: [React Query / Zustand / Context / what exists]

## Commands (repo root)

| Task | Command |
|------|---------|
| Install deps | `[npm ci]` |
| Add an Expo/native package | `[npx expo install package-name]` (picks the SDK-compatible version; ask first) |
| Dev server | `[npx expo start]` (dev build or Expo Go, whichever the repo uses) |
| Lint | `[npx expo lint]` |
| Type check | `[npx tsc --noEmit]` |
| Tests | `[npx jest]` |
| Dependency health | `[npx expo-doctor]` |
| Native build | `[eas build --profile development --platform all]` (ask first) |
| OTA update / store submit | `[eas update]` / `[eas submit]` (ask first, never on your own) |

A change is not done until lint, type check and tests pass and the app starts with `npx expo start` without new red-screen or console errors.

## Layout

- `app/` — Expo Router routes (`_layout.tsx`, `(tabs)/`, `[id].tsx`); keep non-route code out of it
- `components/`, `hooks/`, `lib/` or `src/` — shared UI, hooks and API clients (follow what exists)
- `assets/` — images and fonts · `constants/` — theme and config
- `app.json` / `app.config.ts` — app config, permissions, plugins · `eas.json` — build profiles
- Do not edit generated `ios/` or `android/` folders by hand in a managed (CNG) project; change config or a config plugin instead. Do not touch `node_modules/`, `.expo/` or `dist/`.

## Conventions

1. **Follow the existing structure.** Match current folders, naming and patterns. Don't add a new navigation library, state manager or styling system the repo doesn't already use.
2. **Expo Router for navigation.** Add screens as files in `app/`, use `<Link>` and `useRouter`, and type route params. Don't add React Navigation stacks alongside Expo Router.
3. **Use `npx expo install` for packages.** It pins versions compatible with the SDK; plain `npm install` can pull a mismatched native module.
4. **Platform differences on purpose.** Use `Platform.select`, `.ios.tsx`/`.android.tsx` files or the repo's pattern, and say in the PR which platforms you checked.
5. **Lists.** Use `FlatList`/`FlashList` (whichever exists) with a stable `keyExtractor` for long or dynamic lists, not `.map()` inside a `ScrollView`.
6. **Safe areas and keyboard.** Respect safe-area insets and make sure inputs aren't hidden by the keyboard.
7. **Styles.** Use `StyleSheet.create` or the repo's styling approach; no inline magic numbers when the theme has a token.
8. **Data fetching in hooks/services.** Screens call hooks or services, not raw `fetch` scattered in components. Handle loading, empty, error and offline states.
9. **TypeScript strict, no `any`.** Don't silence errors with `@ts-ignore` or `as any`.
10. **Secrets and storage.** Anything in `app.json`, `app.config.ts` or `EXPO_PUBLIC_*` env vars ships inside the app bundle, so it is public. Keep real secrets on a server. Store tokens with `expo-secure-store`, not `AsyncStorage`.
11. **Permissions.** Request only what the feature needs, at the moment it's needed, and add the usage description strings in app config. Ask before adding a new permission.
12. **Accessible by default.** Give touchables an `accessibilityLabel` and `accessibilityRole`, keep tap targets at least about 44pt, and support larger text sizes.
13. **Dependencies and SDK.** Ask before adding, removing or upgrading a package, or bumping the Expo SDK.

## Testing

- Use React Native Testing Library: render the component, query by role, label or text, and fire events like a user would.
- Mock native modules and network calls at the boundary; don't snapshot whole screens as the only test.
- New behavior gets a test; a bug fix gets a test that fails before the fix where practical.

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by refactors or reformatting of untouched files
- [ ] Lint, `tsc --noEmit` and tests pass; `npx expo-doctor` shows no new problems
- [ ] Checked on [iOS and Android] (simulator, device or dev build) and said which in the PR
- [ ] No `any`, `@ts-ignore`, or secrets in app config or `EXPO_PUBLIC_*` vars
- [ ] Packages added with `npx expo install`; no hand edits to generated native folders
- [ ] Loading, empty, error and offline states handled; long lists virtualized with stable keys
- [ ] New UI has accessibility labels and works with larger text
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify)

## Boundaries — ask first

- Adding a dependency, a native module, a config plugin or a new permission
- Bumping the Expo SDK or React Native version, or changing `app.json`/`app.config.ts`, `eas.json`, bundle IDs or signing
- Running `eas build`, `eas update` or `eas submit`, or anything that ships to users or app stores

## How to customize

1. Fill `[BRACKETED]` values and delete options you don't use (web, bare workflow, test runner).
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
