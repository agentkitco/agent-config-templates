# AGENTS.md — Chrome / browser extension (Manifest V3 + TypeScript)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** a Manifest V3 browser extension written in TypeScript, bundled with [Vite / WXT / Plasmo / webpack], tested with [Vitest] and [Playwright], targeting [Chrome + Edge] and optionally [Firefox]. Replace bracketed defaults if your setup differs, and check the current Chrome extension and store policy docs, since MV3 APIs and review rules change.

## Project

- Extension: [NAME] — [one line on what it does for the user]
- Targets: [Chrome Web Store, Edge Add-ons, Firefox AMO] · minimum browser version: [Chrome 120]
- Surfaces used: [service worker background, content scripts, popup, options page, side panel, offscreen document]
- Data it touches: [which sites, what user data] · where it is stored: [`chrome.storage.local` / `sync` / your backend]
- Privacy policy URL: [link] — store listings require it once you handle user data

## Commands

| Task | Command |
|------|---------|
| Install | `[pnpm install]` |
| Dev build with reload | `[pnpm dev]` → load `[dist/ or .output/chrome-mv3/]` via chrome://extensions → Load unpacked |
| Production build | `[pnpm build]` |
| Firefox build | `[pnpm build --browser firefox]` |
| Typecheck | `[pnpm tsc --noEmit]` |
| Lint | `[pnpm lint]` |
| Unit tests | `[pnpm test]` |
| E2E tests (loads the built extension) | `[pnpm test:e2e]` |
| Package zip | `[pnpm zip]` |
| Publish to a store | **never run by the agent** — a human uploads and submits |

A change is not done until typecheck, lint and tests pass and a production build loads unpacked with no errors on chrome://extensions (service worker and pages).

## Layout

- `[src/]manifest.json` or `[wxt.config.ts]` — the manifest is the source of truth for permissions
- `src/background/` — service worker (event-driven, no DOM)
- `src/content/` — content scripts injected into pages
- `src/popup/`, `src/options/`, `src/sidepanel/` — extension UI pages
- `src/lib/` — shared typed helpers: messaging, storage, API client
- `public/` or `assets/` — icons and static files
- Never commit built `dist/`, store zips, `.pem` signing keys or API secrets.

## Conventions

1. **Permissions are a product decision.** Do not add to `permissions`, `host_permissions`, `optional_permissions` or `content_scripts.matches` unless the task requires it. Prefer `activeTab` and optional permissions requested at the moment of use over broad `<all_urls>`. Call out any manifest permission change in the PR — it can trigger extra store review and a scary update prompt for every user.
2. **The service worker is not a long-lived page.** It can stop at any time. No global state you can't rebuild; persist what matters in `chrome.storage`. Register listeners at top level, synchronously. Use `chrome.alarms` instead of `setInterval` for anything periodic.
3. **No remote code.** MV3 forbids loading or `eval`-ing remote JavaScript. No `eval`, `new Function`, remote `<script>` tags or string-built code. Fetching data (JSON) is fine; fetching code is not.
4. **Typed messaging.** Every message between content script, background and UI pages has a typed shape in `src/lib/messages.ts`. Validate messages from content scripts in the background — they run inside pages you don't control.
5. **Content scripts are guests.** Keep them small. Don't leak globals or override page styles; scope CSS (Shadow DOM or prefixed classes). Never insert untrusted text with `innerHTML`; use `textContent` or a sanitizer. Don't trust anything read from the page.
6. **Storage on purpose.** `storage.sync` has small quotas — use it only for settings. Version stored data and migrate it in `runtime.onInstalled` rather than breaking existing users.
7. **Secrets don't belong in the bundle.** Anything shipped in the extension is readable by anyone. API keys that must stay secret go behind your own backend.
8. **Async APIs, handled errors.** Use the promise forms of `chrome.*` APIs, check `chrome.runtime.lastError` where callbacks remain, and handle "Receiving end does not exist" when a tab has no content script.
9. **Cross-browser through a wrapper.** If you target Firefox, use [`webextension-polyfill` / the framework's `browser` global] and keep browser-specific code behind small checks.
10. **Accessible UI.** Popup and options pages are keyboard-usable, labelled, and readable at 200% zoom.

## Testing

- Unit-test pure logic (parsing, message handlers, storage migrations) with `chrome.*` mocked.
- E2E: load the built extension in Playwright (persistent context with `--load-extension`) and cover the main user flow on a local test page, not a live third-party site.
- For a manifest change, rebuild and confirm the extension loads with no errors and the permission prompt shows what you expect.

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by refactors or dependency bumps
- [ ] No new permissions, host permissions or content-script matches, or each one is justified in the PR
- [ ] No remote code, `eval`, or unsanitized `innerHTML`
- [ ] Service worker survives a restart (state persisted, listeners top-level)
- [ ] Stored data stays compatible, or a migration is included
- [ ] Typecheck, lint, tests and production build pass; loads unpacked with no errors
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify)

## Boundaries — ask first

- Uploading, submitting or publishing to any store, or changing store listing text, screenshots or privacy disclosures
- Adding or widening permissions, host permissions or `externally_connectable`
- Changing the extension `key`, ID, signing setup or update URL
- Collecting new user data, adding analytics or sending data to a new domain
- Removing features existing users rely on, or changes that wipe stored user data
- Major upgrades of the build framework or adding large dependencies

## How to customize

1. Fill `[BRACKETED]` values and delete options you don't use (bundler, Firefox, surfaces).
2. Delete sections that do not apply (keep the file short — agents follow short files better).
3. Optionally add a one-line `CLAUDE.md` with `@AGENTS.md` so Claude Code shares this file.

---

### Disclaimer

Written with AI assistance by [AgentKit Co](https://agentkitco.com/). Starting point, not a guarantee — review against your project, the current Chrome Web Store policies and your own release checks.

### Want more?

- Free templates + generator: https://agentkitco.com/tools/agents-md-generator/
- [AI Coding Agent Config Kit](https://agentkitco.com/products/agent-config-kit/) ($14) — 12-stack AGENTS.md set, Cursor rules, Copilot files
- [AI PR Review Kit](https://agentkitco.com/products/ai-pr-review-kit/) ($17) — blockers-first checklists, paste-ready PR comments, review prompts for Claude Code & Cursor

License: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — use and adapt with attribution to AgentKit Co.
