<!-- Free template from AgentKit Co (https://github.com/agentkitco/agent-config-templates, CC BY 4.0). Save as .github/copilot-instructions.md and delete this comment. More Copilot files are in the AI Coding Agent Config Kit ($14): https://agentkitco.com/products/agent-config-kit/ -->
# Project instructions

## Stack
- [LANGUAGE + FRAMEWORK, e.g. Next.js App Router + TypeScript], package manager [pnpm]. Never use [npm or yarn].

## Commands
- Install: `[pnpm install]`
- Test: `[pnpm test]`. Lint: `[pnpm lint]`. Types: `[pnpm typecheck]`.
- Run the tests and lint after every change and fix failures before you finish.

## Layout
- `[src/]` application code. `[tests/]` tests. `[src/generated/]` is rebuilt by `[pnpm codegen]`; never edit it by hand.

## Conventions
- [ONE RULE PER LINE: the mistakes you keep correcting, not general advice.]
- Ask before adding a new dependency.
- Never put secrets, tokens or API keys in code, logs or client-side config.

## Code review
- When performing a code review, flag any SQL or shell command built with string concatenation.
- When performing a code review, check that every behavior change has a test.
- When performing a code review, don't comment on formatting; the linter handles it.
