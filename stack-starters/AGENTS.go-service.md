# AGENTS.md — Go service / CLI

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** Go [1.22+], modules (`go.mod`), `cmd/` + `internal/`, `go test`, `golangci-lint` optional. Replace bracketed defaults if your project differs.

## Project

- App: [PROJECT NAME] — [one-line purpose]
- Runtime: Go [1.22+] · Module path: `[example.com/project]`
- Shape: [HTTP service / CLI / both]

## Commands (repo root)

| Task | Command |
|------|---------|
| Download deps | `go mod download` |
| Tidy | `go mod tidy` |
| Run (main) | `go run ./cmd/[app]/` |
| Build | `go build -o bin/[app] ./cmd/[app]/` |
| Unit tests | `go test ./...` · one package: `go test ./internal/[pkg]/` · race: `go test -race ./...` |
| Vet | `go vet ./...` |
| Lint (if configured) | `golangci-lint run` |

A change is not done until `go test` (and `go vet`) for touched packages pass. Prefer the narrowest package while iterating; run `./...` before handing off.

## Layout

- `cmd/[app]/` — `main` packages only (thin; wire dependencies here)
- `internal/` — app code not importable by other modules
- `pkg/` — only if you intentionally expose a library API
- `api/` or `docs/` — OpenAPI / prose contracts when present
- Tests beside source: `*_test.go` (table-driven preferred)

## Conventions

1. **Accept `context.Context` as the first parameter** on I/O, RPCs, and long-running work. Never store a context on a struct.
2. **Errors.** Return `error`; wrap with `%w` and a short what/why. Compare with `errors.Is` / `errors.As`. Do not panic for expected failure.
3. **Interfaces.** Define small interfaces at the **consumer** package, not the implementer.
4. **Config.** Load from env / flags once at startup; validate required values before serving. Never log secrets.
5. **HTTP (if applicable).** Timeouts on clients and servers; validate request bodies; map domain errors to stable status codes; no bare `http.DefaultClient` for outbound calls.
6. **Concurrency.** Prefer `errgroup` or clear ownership of goroutines; document who cancels what. No unprotected shared maps.
7. **Dependencies.** Prefer stdlib. Ask before adding a module. Pin via `go.mod` / `go.sum` only through `go get` / `go mod tidy`.

## Review habits (before you say “done”)

- [ ] Diff is scoped to the ask — no drive-by refactors
- [ ] New exported APIs have a clear consumer and docs comment when public
- [ ] Errors wrap with context; no swallowed `err`
- [ ] No secrets, tokens, or PII in logs or commits
- [ ] `go test` (and race where concurrency changed) and `go vet` pass on touched packages
- [ ] If you opened a PR: blockers-first summary (what broke if wrong → what changed → how to verify)

## Boundaries — ask first

- Adding a module dependency, changing Go version, CI, auth, or datastore schema
- Deleting public packages, rewriting git history, force-pushing, or touching production infra

## Never

- Commit `vendor/` unless the repo already vendors on purpose
- Edit `go.sum` by hand or leave the module graph untidy
- Claim a task is done when tests or vet failed
- Use `init()` for non-trivial wiring (prefer explicit `main` / constructors)

## How to customize

1. Fill `[BRACKETED]` values.
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
