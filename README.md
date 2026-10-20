# Agent config templates

Six free, copy-paste starting points for telling AI coding agents how your project works: two `AGENTS.md` templates, one Cursor rule, and three prompts that draft, audit and test an instruction file.

| File | What it is |
|---|---|
| [templates/AGENTS.nextjs-typescript.md](templates/AGENTS.nextjs-typescript.md) | `AGENTS.md` for a Next.js (App Router) + TypeScript app |
| [templates/AGENTS.python-package.md](templates/AGENTS.python-package.md) | `AGENTS.md` for a Python package or script collection with pytest |
| [cursor-rules/project-baseline.mdc](cursor-rules/project-baseline.mdc) | Always-on Cursor project rule (`.cursor/rules/`) |
| [prompts/draft-agents-md-from-repo.md](prompts/draft-agents-md-from-repo.md) | Prompt: have an agent propose an `AGENTS.md` from your repo |
| [prompts/audit-and-trim-instruction-file.md](prompts/audit-and-trim-instruction-file.md) | Prompt: audit and trim an instruction file that has grown noisy |
| [prompts/test-with-five-probe-tasks.md](prompts/test-with-five-probe-tasks.md) | Prompt: test whether the file actually works |

## How to use

1. Copy a template into your repo (for example `AGENTS.md` at the root, or `.cursor/rules/project-baseline.mdc`).
2. Replace every `[BRACKETED]` value with your real commands, paths and versions. The defaults are common commands and may be wrong for your project.
3. If you also use Claude Code, a short `CLAUDE.md` containing `@AGENTS.md` imports the same file (see Anthropic's Claude Code memory docs). Check each tool's current documentation, since formats change.

## Honest notes

- These templates were written with AI assistance. They are starting points, not tested guarantees. Review them and try them on your own project.
- Instruction files are guidance for an agent, not enforcement. Keep real checks in CI.
- AgentKit Co is not affiliated with or endorsed by Anthropic, Cursor, GitHub, Microsoft or OpenAI.

## More

- Free in-browser generator that outputs `AGENTS.md`, `CLAUDE.md`, a Cursor rule and a Copilot file for your stack: <https://agentkitco.com/tools/agents-md-generator/>
- Maintained by [AgentKit Co](https://agentkitco.com). A longer paid kit is available as the [AI Coding Agent Config Kit](https://agentkitco.com/products/agent-config-kit/) ($14): a 34-page PDF plus a drop-in ZIP of 54 files (AGENTS.md templates for 12 stacks, 8 Cursor `.mdc` rules, 3 Claude Code rule files, 3 Copilot files and 12 audit/test prompts). Everything in this repo is free and nothing here is held back from it.

## License

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Use, share and adapt with attribution to AgentKit Co.
