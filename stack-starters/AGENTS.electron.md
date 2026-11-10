# AGENTS.md — Electron (TypeScript desktop app)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** Electron [33+] desktop app in TypeScript, built with [Electron Forge / electron-builder / electron-vite], renderer UI in [React / Vue / Svelte / plain HTML] bundled by [Vite / webpack], package manager [npm / pnpm / yarn], tests with [Vitest / Jest] and [Playwright for Electron] end-to-end. Replace bracketed defaults if your project differs, and check Electron's current docs and security checklist for your version, since defaults, APIs and release cadence change often.

## Project

- App: [PROJECT NAME] — [one-line purpose]
- Platforms: [macOS / Windows / Linux] · app ID: [com.example.app]
- Processes: main (`[src/main/]`), preload (`[src/preload/]`), renderer (`[src/renderer/]`)
- Auto-update: [electron-updater / update.electronjs.org / none] · backend/API: [base URL or none]

## Commands (repo root)

| Task | Command |
|------|---------|
| Install | `[npm ci]` |
| Dev (hot reload) | `[npm run dev]` / `[npm start]` |
| Type check | `[npm run typecheck]` (`tsc --noEmit` for main, preload and renderer) |
| Lint / format | `[npm run lint]` / `[npm run format:check]` |
| Unit tests | `[npm test]` |
| E2E tests | `[npm run test:e2e]` (Playwright `_electron.launch`) |
| Package (local, unsigned) | `[npm run package]` |
| Make installers / publish | `[npm run make]` / `[npm run publish]` (ask first) |

A change is not done until type check, lint and unit tests pass and the app starts in dev without new console errors in either the main process or DevTools. Run E2E tests when you touched windows, menus, IPC or startup, and say in the PR if you could not.

## Layout

- `src/main/` — app lifecycle, `BrowserWindow` creation, menus, tray, IPC handlers, native/OS access
- `src/preload/` — the only bridge: `contextBridge.exposeInMainWorld` with a small typed API
- `src/renderer/` — UI only; treat it like an untrusted web page
- `src/shared/` — [IPC channel names and types shared by all three]
- `forge.config.*` / `electron-builder.*` / `electron.vite.config.*` — packaging and build config
- Do not edit `out/`, `dist/`, `release/` or `node_modules/` by hand.

## Conventions

1. **Follow the existing structure.** Match current folders, build tool and UI framework. Don't add a second bundler, state library or IPC wrapper.
2. **Keep the security defaults on.** `contextIsolation: true`, `nodeIntegration: false`, `sandbox: true`, `webSecurity: true`. Never turn these off to make something work; route it through preload and IPC instead. Never enable the `remote` module.
3. **Narrow, typed IPC.** Expose specific functions from preload (`api.saveFile(data)`), never raw `ipcRenderer` or a generic `send(channel, ...)`. Use `ipcRenderer.invoke` / `ipcMain.handle` for request/response, keep channel names in one shared file, and validate every argument in the main process as if it came from the internet.
4. **Check the sender.** In IPC handlers, verify `event.senderFrame` belongs to your app's origin before doing file, shell or network work.
5. **Lock down navigation.** Set a Content-Security-Policy, block unexpected `will-navigate`, and use `setWindowOpenHandler` to deny new windows or send allowed links to `shell.openExternal` after checking the URL (https and allow-listed hosts only).
6. **No Node in the renderer.** The renderer does not import `fs`, `child_process`, `path` or native modules; ask main through preload.
7. **Don't block the main process.** No sync file or network I/O, `execSync` or heavy loops in main; it freezes every window. Use async APIs, worker threads or a utility process.
8. **Window lifecycle.** Create windows after `app.whenReady()`, handle `window-all-closed` and `activate` per platform (macOS keeps the app alive), clean up listeners when windows close, and use `requestSingleInstanceLock` if the app should run once.
9. **Paths and files.** Use `app.getPath('userData')` and friends for app data, `path.join` for paths, and never write inside the app bundle or `asar`. Remember packaged paths differ from dev (`app.isPackaged`, `process.resourcesPath`).
10. **Cross-platform.** Don't hard-code `/` or `\`, shell commands, or keyboard shortcuts; use `CommandOrControl` accelerators and check menu roles on macOS vs Windows/Linux.
11. **Secrets and storage.** Anything shipped in the app can be extracted, so treat it as public. Keep real secrets on a server; store user tokens with `safeStorage` or the OS keychain, not plain JSON or `localStorage`. Never commit signing certificates, `.p12` files, Apple IDs or notarization passwords.
12. **Native modules.** Rebuild native dependencies for Electron's Node ABI (the build tool usually does this); if a native module breaks, say so instead of swapping in a different library.

## Testing

- Unit-test pure logic, IPC argument validation and renderer components without launching Electron.
- E2E tests launch the built app with Playwright's `_electron`, wait for the first window, and assert visible state by role or test ID; avoid fixed sleeps.
- New behavior gets a test; a bug fix gets a test that fails before the fix where practical. Note which OS you tested on.

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by refactors, reformatting or build-config churn
- [ ] Type check, lint and unit tests pass; E2E run or noted as not run; app starts with no new errors
- [ ] `contextIsolation`, `sandbox` and `nodeIntegration: false` unchanged; CSP still present
- [ ] New IPC is a named, typed function with argument validation and a sender check in main
- [ ] No sync or heavy work added to the main process; listeners cleaned up on window close
- [ ] External links and new windows go through the open handler and an allow-list
- [ ] Checked on [OS] and said which in the PR; no secrets, certificates or signing credentials committed
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify)

## Boundaries — ask first

- Upgrading Electron or the build tool, or adding, removing or upgrading native modules
- Changing `webPreferences`, the CSP, protocol handlers, deep links or file associations
- Changing app ID, signing, notarization, entitlements, installer or auto-update settings
- Running `make`/`publish`, uploading releases, or anything that ships to users

## How to customize

1. Fill `[BRACKETED]` values and delete options you don't use (build tool, UI framework, test runner, auto-update).
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
