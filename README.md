# Agent config templates

Twenty-five free, copy-paste starting points for telling AI coding agents how your project works: two short `AGENTS.md` templates, seventeen polished stack starters, one Cursor rule, two GitHub Copilot files, and three prompts that draft, audit and test an instruction file.

| File | What it is |
|---|---|
| [templates/AGENTS.nextjs-typescript.md](templates/AGENTS.nextjs-typescript.md) | `AGENTS.md` for a Next.js (App Router) + TypeScript app |
| [templates/AGENTS.python-package.md](templates/AGENTS.python-package.md) | `AGENTS.md` for a Python package or script collection with pytest |
| [stack-starters/AGENTS.nextjs-typescript.md](stack-starters/AGENTS.nextjs-typescript.md) | Polished Next.js App Router + TypeScript `AGENTS.md` (commands, review habits, soft CTA) |
| [stack-starters/AGENTS.python-fastapi.md](stack-starters/AGENTS.python-fastapi.md) | Polished Python FastAPI `AGENTS.md` (routers, pydantic, review habits, soft CTA) |
| [stack-starters/AGENTS.go-service.md](stack-starters/AGENTS.go-service.md) | Polished Go service/CLI `AGENTS.md` (modules, context, review habits, soft CTA) |
| [stack-starters/AGENTS.rust-axum.md](stack-starters/AGENTS.rust-axum.md) | Polished Rust Axum HTTP service/CLI `AGENTS.md` (Cargo, clippy, review habits, soft CTA) |
| [stack-starters/AGENTS.nestjs.md](stack-starters/AGENTS.nestjs.md) | Polished NestJS (Node.js + TypeScript) `AGENTS.md` (modules, DTOs, review habits, soft CTA) |
| [stack-starters/AGENTS.django.md](stack-starters/AGENTS.django.md) | Polished Django (Python) `AGENTS.md` (apps, migrations, ORM, review habits, soft CTA) |
| [stack-starters/AGENTS.rails.md](stack-starters/AGENTS.rails.md) | Polished Ruby on Rails `AGENTS.md` (migrations, strong params, N+1, review habits, soft CTA) |
| [stack-starters/AGENTS.laravel.md](stack-starters/AGENTS.laravel.md) | Polished Laravel (PHP) `AGENTS.md` (Eloquent, Form Requests, migrations, review habits, soft CTA) |
| [stack-starters/AGENTS.spring-boot.md](stack-starters/AGENTS.spring-boot.md) | Polished Spring Boot (Java) `AGENTS.md` (Maven/Gradle, DTOs, transactions, Flyway, review habits, soft CTA) |
| [stack-starters/AGENTS.aspnet-core.md](stack-starters/AGENTS.aspnet-core.md) | Polished ASP.NET Core (C# / .NET) `AGENTS.md` (dotnet CLI, EF Core migrations, DI lifetimes, async, review habits, soft CTA) |
| [stack-starters/AGENTS.nuxt.md](stack-starters/AGENTS.nuxt.md) | Polished Nuxt (Vue 3) `AGENTS.md` (app/pages, Nitro server/api, useFetch, review habits, soft CTA) |
| [stack-starters/AGENTS.sveltekit.md](stack-starters/AGENTS.sveltekit.md) | Polished SvelteKit (Svelte 5) `AGENTS.md` (routes, load/actions, runes, $lib/server, review habits, soft CTA) |
| [stack-starters/AGENTS.express-typescript.md](stack-starters/AGENTS.express-typescript.md) | Polished Express (Node.js + TypeScript) `AGENTS.md` (thin handlers, zod validation, async errors, middleware order, supertest, review habits, soft CTA) |
| [stack-starters/AGENTS.flask.md](stack-starters/AGENTS.flask.md) | Polished Flask (Python) `AGENTS.md` (app factory, blueprints, Flask-Migrate, CSRF, Jinja escaping, pytest, review habits, soft CTA) |
| [stack-starters/AGENTS.hono.md](stack-starters/AGENTS.hono.md) | Polished Hono (TypeScript) `AGENTS.md` (Workers/Node/Deno/Bun, zValidator, Bindings, app.request tests, review habits, soft CTA) |
| [stack-starters/AGENTS.astro.md](stack-starters/AGENTS.astro.md) | Polished Astro (TypeScript) `AGENTS.md` (zero-JS islands, content collections, astro:assets, astro check/build, review habits, soft CTA) |
| [stack-starters/AGENTS.phoenix.md](stack-starters/AGENTS.phoenix.md) | Polished Phoenix (Elixir) `AGENTS.md` (contexts, changesets, Ecto migrations, LiveView streams, verified routes, ExUnit, review habits, soft CTA) |
| [cursor-rules/project-baseline.mdc](cursor-rules/project-baseline.mdc) | Always-on Cursor project rule (`.cursor/rules/`) |
| [copilot/copilot-instructions.md](copilot/copilot-instructions.md) | Repository-wide GitHub Copilot instructions (`.github/copilot-instructions.md`), including code review rules |
| [copilot/tests.instructions.md](copilot/tests.instructions.md) | Path-specific Copilot instructions with an `applyTo` glob (`.github/instructions/`) |
| [prompts/draft-agents-md-from-repo.md](prompts/draft-agents-md-from-repo.md) | Prompt: have an agent propose an `AGENTS.md` from your repo |
| [prompts/audit-and-trim-instruction-file.md](prompts/audit-and-trim-instruction-file.md) | Prompt: audit and trim an instruction file that has grown noisy |
| [prompts/test-with-five-probe-tasks.md](prompts/test-with-five-probe-tasks.md) | Prompt: test whether the file actually works |

## How to use

1. Copy a template into your repo (for example `AGENTS.md` at the root, `.cursor/rules/project-baseline.mdc`, or `.github/copilot-instructions.md`).
2. Replace every `[BRACKETED]` value with your real commands, paths and versions. The defaults are common commands and may be wrong for your project.
3. If you also use Claude Code, a short `CLAUDE.md` containing `@AGENTS.md` imports the same file (see Anthropic's Claude Code memory docs). Check each tool's current documentation, since formats change.

## If the agent ignores your file

Free troubleshooting guides and examples from AgentKit Co:

- [AGENTS.md not working? Fixes](https://agentkitco.com/blog/agents-md-not-working/)
- [CLAUDE.md not working? Fixes](https://agentkitco.com/blog/claude-md-not-working/)
- [Cursor rules not working? Fixes](https://agentkitco.com/blog/cursor-rules-not-working/)
- [Copilot ignoring `copilot-instructions.md`? Fixes](https://agentkitco.com/blog/copilot-instructions-not-working/)
- [AI coding agent stuck in a loop? How to break it](https://agentkitco.com/blog/ai-coding-agent-stuck-in-loop/)
- [Still on `.cursorrules`? Migrate it to `.mdc` project rules](https://agentkitco.com/blog/migrate-cursorrules-to-mdc/)
- [Reviewing what the agent wrote? A 10-step checklist for AI-generated pull requests](https://agentkitco.com/blog/review-ai-generated-pull-requests/)
- Worked examples: [AGENTS.md](https://agentkitco.com/blog/agents-md-examples/), [CLAUDE.md](https://agentkitco.com/blog/claude-md-examples/), [Cursor rules](https://agentkitco.com/blog/cursor-rules-examples/), [copilot-instructions.md](https://agentkitco.com/blog/copilot-instructions-md-examples/)

## Honest notes

- These templates were written with AI assistance. They are starting points, not tested guarantees. Review them and try them on your own project.
- Instruction files are guidance for an agent, not enforcement. Keep real checks in CI.
- AgentKit Co is not affiliated with or endorsed by Anthropic, Cursor, GitHub, Microsoft or OpenAI.


## Free stack AGENTS.md

Ready-to-drop starters (longer than the short bracket templates above). Copy a file to your repo root as `AGENTS.md`, fill the `[BRACKETED]` bits, and trim what you do not need.

| Stack | File |
|---|---|
| Next.js App Router + TypeScript | [stack-starters/AGENTS.nextjs-typescript.md](stack-starters/AGENTS.nextjs-typescript.md) |
| Python FastAPI + pydantic v2 | [stack-starters/AGENTS.python-fastapi.md](stack-starters/AGENTS.python-fastapi.md) |
| Go service / CLI (modules) | [stack-starters/AGENTS.go-service.md](stack-starters/AGENTS.go-service.md) |
| Rust Axum HTTP service / CLI | [stack-starters/AGENTS.rust-axum.md](stack-starters/AGENTS.rust-axum.md) |
| NestJS (Node.js + TypeScript) | [stack-starters/AGENTS.nestjs.md](stack-starters/AGENTS.nestjs.md) |
| Django (Python, optional DRF) | [stack-starters/AGENTS.django.md](stack-starters/AGENTS.django.md) |
| Ruby on Rails (7.x / 8.x) | [stack-starters/AGENTS.rails.md](stack-starters/AGENTS.rails.md) |
| Laravel (PHP 8.2+, 11.x / 12.x) | [stack-starters/AGENTS.laravel.md](stack-starters/AGENTS.laravel.md) |
| Spring Boot 3.x (Java 17 / 21) | [stack-starters/AGENTS.spring-boot.md](stack-starters/AGENTS.spring-boot.md) |
| ASP.NET Core (.NET 8 / 9, C#) | [stack-starters/AGENTS.aspnet-core.md](stack-starters/AGENTS.aspnet-core.md) |
| Nuxt 4 / 3 (Vue 3, Nitro) | [stack-starters/AGENTS.nuxt.md](stack-starters/AGENTS.nuxt.md) |
| SvelteKit 2 (Svelte 5 runes) | [stack-starters/AGENTS.sveltekit.md](stack-starters/AGENTS.sveltekit.md) |
| Express (Node.js + TypeScript) | [stack-starters/AGENTS.express-typescript.md](stack-starters/AGENTS.express-typescript.md) |
| Flask (Python, app factory) | [stack-starters/AGENTS.flask.md](stack-starters/AGENTS.flask.md) |
| Hono 4 (TypeScript, Workers / Node / Deno / Bun) | [stack-starters/AGENTS.hono.md](stack-starters/AGENTS.hono.md) |
| Astro 5 (TypeScript, static or server) | [stack-starters/AGENTS.astro.md](stack-starters/AGENTS.astro.md) |
| Phoenix 1.7 / 1.8 (Elixir, Ecto, LiveView) | [stack-starters/AGENTS.phoenix.md](stack-starters/AGENTS.phoenix.md) |

Need a blockers-first review after the agent opens a PR? The [AI PR Review Kit](https://agentkitco.com/products/ai-pr-review-kit/) ($17) pairs with these starters. More stacks and Cursor/Copilot files: [AI Coding Agent Config Kit](https://agentkitco.com/products/agent-config-kit/) ($14). Site: <https://agentkitco.com/>.

## More

- Free in-browser generator that outputs `AGENTS.md`, `CLAUDE.md`, a Cursor rule and a Copilot file for your stack: <https://agentkitco.com/tools/agents-md-generator/>
- Maintained by [AgentKit Co](https://agentkitco.com). A longer paid kit is available as the [AI Coding Agent Config Kit](https://agentkitco.com/products/agent-config-kit/) ($14): a 34-page PDF plus a drop-in ZIP of 54 files (AGENTS.md templates for 12 stacks, 8 Cursor `.mdc` rules, 3 Claude Code rule files, 3 Copilot files and 12 audit/test prompts). Everything in this repo is free and nothing here is held back from it.
- For AI-written pull requests, the [AI PR Review Kit](https://agentkitco.com/products/ai-pr-review-kit/) ($17) includes blockers-first checklists, six paste-ready PR comments, 12 review prompts and a read-only reviewer agent for Claude Code and Cursor. Pairs with the free [review checklist](https://agentkitco.com/blog/review-ai-generated-pull-requests/) above.

## License

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Use, share and adapt with attribution to AgentKit Co.
