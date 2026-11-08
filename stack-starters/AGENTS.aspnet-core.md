# AGENTS.md — ASP.NET Core (C# / .NET)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** .NET [8 / 9], ASP.NET Core [Minimal APIs / MVC controllers], persistence via [Entity Framework Core / Dapper], tests via [xUnit / NUnit] + [FluentAssertions / Moq / NSubstitute]. Replace bracketed defaults if your project differs.

## Project

- App: [PROJECT NAME] — [one-line purpose]
- Runtime: .NET [8 / 9] · ASP.NET Core [Minimal APIs / controllers]
- Database: [SQL Server / PostgreSQL / SQLite for local] · migrations via [EF Core migrations]
- Solution: `[App.sln]` with projects `[src/App.Api]`, `[src/App.Core]`, `[tests/App.Tests]`
- Formatting: `dotnet format` with the repo's `.editorconfig`

## Commands (repo root)

| Task | Command |
|------|---------|
| Restore | `dotnet restore` |
| Build | `dotnet build --no-restore` |
| Run locally | `dotnet run --project [src/App.Api]` |
| All tests | `dotnet test` |
| One test | `dotnet test --filter "FullyQualifiedName~[OrderServiceTests]"` |
| Format check | `dotnet format --verify-no-changes` |
| Add migration | `dotnet ef migrations add [Name] --project [src/App.Infrastructure] --startup-project [src/App.Api]` |

A change is not done until the narrowest relevant tests pass, then `dotnet build` (with warnings as errors if the repo sets `TreatWarningsAsErrors`) and `dotnet test` pass before handing off.

## Layout

- `src/[App.Api]/` — `Program.cs`, endpoints or controllers, DI registration, middleware
- `src/[App.Core]/` — domain types and business logic; no ASP.NET or EF references if the repo keeps it that way
- `src/[App.Infrastructure]/` — `DbContext`, EF configurations, migrations, external clients
- `appsettings.json` (+ `appsettings.[Environment].json`) — configuration; no secrets
- `tests/[App.Tests]/` — unit and integration tests mirroring the source structure

## Conventions

1. **Follow the existing structure.** Match the project split, naming, and patterns already in the solution. Don't introduce MediatR, AutoMapper, a repository layer over EF, or Clean Architecture folders unless the codebase already uses them.
2. **Nullable and async.** Respect `<Nullable>enable</Nullable>`; don't silence warnings with `!` without a reason. Use `async`/`await` end to end, pass `CancellationToken` through, and never block with `.Result` or `.Wait()`.
3. **Dependency injection.** Register services in the existing composition root with the right lifetime. Never inject a scoped service (like `DbContext`) into a singleton.
4. **DTOs at the edge.** Endpoints accept and return request/response records, never EF entities. Validate input the way peers do ([data annotations / FluentValidation]) and return errors as `ProblemDetails` consistent with existing handlers.
5. **EF Core.** Use `AsNoTracking()` for read-only queries, project to DTOs with `Select` for lists, and watch for N+1 (use `Include` or projections). Use LINQ or parameterized `FromSql`; never concatenate user input into SQL.
6. **Migrations.** Every model change ships with a new EF migration in the same diff; review the generated SQL for data loss. Never edit a migration that already ran in a shared environment.
7. **Config and secrets.** Bind config with the Options pattern (`IOptions<T>`). Secrets come from user secrets locally and environment variables or the vault in deployed environments, never committed `appsettings*.json`.
8. **Security.** New endpoints follow the existing auth setup (`RequireAuthorization`, policies, `[Authorize]`). Don't add `AllowAnonymous` or widen CORS without a one-line why.
9. **Logging.** Use `ILogger<T>` with structured message templates (`"Order {OrderId} created"`), not string interpolation. Don't log tokens, passwords, or PII.
10. **Dependencies.** Ask before adding a NuGet package or changing the target framework. Respect central package management (`Directory.Packages.props`) if the repo uses it.

## Testing

- Unit tests for business logic; integration tests with `WebApplicationFactory<Program>` for endpoints.
- Use [Testcontainers] for database tests if the repo already does; don't switch a SQL Server or Postgres project's tests to the EF in-memory provider.
- New behavior gets a test; a bug fix gets a test that fails before the fix.

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by refactors or reformatting of untouched files
- [ ] Model changes include a new migration, and its SQL was checked
- [ ] Endpoints use DTOs + validation; no entities leaked in responses
- [ ] No sync-over-async, no scoped-into-singleton DI, `CancellationToken` passed through
- [ ] Auth and CORS unchanged or deliberately changed with a reason
- [ ] No secrets, tokens, or PII in logs, test fixtures, or commits
- [ ] `dotnet build`, `dotnet test`, and `dotnet format --verify-no-changes` pass
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify)

## Boundaries — ask first

- Adding a NuGet package, changing the target framework or SDK (`global.json`), or editing CI
- Destructive or long-locking migrations, or touching production data
- Changing authentication, authorization policies, CORS, or data protection keys
- Rewriting git history, force-pushing, or changing deploy config (Dockerfile, Helm, Bicep, Azure/AWS settings)

## Never

- Commit secrets, connection strings with passwords, or `appsettings.*.json` holding real credentials
- Edit migrations that already ran in shared environments
- Swallow exceptions with empty `catch` blocks or log full request bodies containing PII
- Claim a task is done when the build or tests failed

## How to customize

1. Fill `[BRACKETED]` values and delete options you don't use.
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
