# AGENTS.md — Spring Boot (Java)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** Spring Boot [3.x], Java [17 / 21], build via [Maven / Gradle], tests via JUnit 5 + [Mockito / Testcontainers], persistence via [Spring Data JPA / JDBC]. Replace bracketed defaults if your project differs.

## Project

- App: [PROJECT NAME] — [one-line purpose]
- Runtime: Java [17 / 21] · Spring Boot [3.x] · [web MVC / WebFlux]
- Database: [PostgreSQL / MySQL / H2 for local] · migrations via [Flyway / Liquibase]
- Build: [Maven wrapper `./mvnw` / Gradle wrapper `./gradlew`]
- Formatting: [Spotless / google-java-format / Checkstyle / none]

## Commands (repo root)

| Task | Maven | Gradle |
|------|-------|--------|
| Build (no tests) | `./mvnw -DskipTests package` | `./gradlew assemble` |
| Run locally | `./mvnw spring-boot:run` | `./gradlew bootRun` |
| All tests | `./mvnw test` | `./gradlew test` |
| One test class | `./mvnw test -Dtest=[FooServiceTest]` | `./gradlew test --tests '[*FooServiceTest]'` |
| Format / lint | `[./mvnw spotless:apply]` | `[./gradlew spotlessApply]` |
| Full check | `./mvnw verify` | `./gradlew check` |

Always use the wrapper (`./mvnw` / `./gradlew`), not a globally installed Maven or Gradle. A change is not done until the narrowest relevant tests pass and the full check passes before handing off.

## Layout

- `src/main/java/[com/example/app]/` — code, organized by [feature package / layer]
- `.../web/` or `.../controller/` — `@RestController` classes; thin, delegate to services
- `.../service/` — business logic, `@Transactional` boundaries live here
- `.../repository/` — Spring Data repositories
- `.../domain/` or `.../entity/` — JPA entities; DTOs/records kept separate from entities
- `src/main/resources/application.yml` (+ `application-[profile].yml`) — configuration
- `src/main/resources/db/migration/` — Flyway migrations (or Liquibase changelogs)
- `src/test/java/` — tests mirroring the main package structure

## Conventions

1. **Follow the existing structure.** Match the package layout, naming, and patterns already in the repo. Don't introduce hexagonal/clean-architecture layers, MapStruct, Lombok, or a new library unless the codebase already uses it.
2. **Constructor injection.** Use constructor injection (final fields), not `@Autowired` on fields.
3. **DTOs at the edge.** Controllers accept and return DTOs or Java records, never JPA entities. Validate input with Bean Validation (`@Valid`, `@NotNull`, `@Size`) and keep error responses consistent with the existing `@ControllerAdvice`.
4. **Transactions.** Put `@Transactional` on service methods, not controllers. Mark read paths `@Transactional(readOnly = true)` where peers do. Don't call a `@Transactional` method from the same class and expect a new transaction.
5. **Queries.** Watch for N+1 on lazy associations — use fetch joins, `@EntityGraph`, or projections for list endpoints. Use parameter binding; never concatenate user input into JPQL/SQL.
6. **Migrations.** Every schema change ships with a new Flyway/Liquibase migration in the same diff. Never edit a migration that already ran in a shared environment. Don't rely on `spring.jpa.hibernate.ddl-auto=update` outside local dev.
7. **Config and secrets.** Read config through `@ConfigurationProperties` or existing patterns. Secrets come from environment variables or the secret store, never committed `application*.yml`.
8. **Security.** New endpoints follow the existing Spring Security config (filter chain, method security, roles). Don't widen `permitAll()` or disable CSRF without a one-line why.
9. **Dependencies.** Ask before adding a dependency or changing the Spring Boot / Java version. Let the Spring Boot BOM manage versions; don't pin versions it already manages.

## Testing

- Unit tests for services with mocks; slice tests (`@WebMvcTest`, `@DataJpaTest`) for controllers and repositories; `@SpringBootTest` only when the full context is really needed (it is slow).
- Use [Testcontainers] for database tests if the repo already does; don't switch a Postgres project's tests to H2.
- New behavior gets a test; a bug fix gets a test that fails before the fix.

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by refactors or reformatting of untouched files
- [ ] Schema changes include a new migration
- [ ] Controllers use DTOs + validation; no entities leaked in responses
- [ ] Transaction boundaries and lazy loading checked (no N+1, no `LazyInitializationException` waiting to happen)
- [ ] Security config unchanged or deliberately changed with a reason
- [ ] No secrets, tokens, or PII in logs, test fixtures, or commits
- [ ] Tests and the full build check pass
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify)

## Boundaries — ask first

- Adding a dependency, upgrading Spring Boot or Java, or changing the build plugins or CI
- Destructive or long-locking migrations, or touching production data
- Changing Spring Security config, CORS, or session handling
- Rewriting git history, force-pushing, or changing deploy config (Dockerfile, Helm, Kubernetes, cloud config)

## Never

- Commit secrets, `.env` files, or real credentials in `application*.yml`
- Edit migrations that already ran in shared environments
- Swallow exceptions silently or log full request bodies containing PII
- Claim a task is done when tests or the build failed

## How to customize

1. Fill `[BRACKETED]` values and delete the build tool column you don't use.
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
