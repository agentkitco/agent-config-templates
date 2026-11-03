---
applyTo: "[**/*.test.ts,**/*.test.tsx]"
---

<!-- Free template from AgentKit Co (https://github.com/agentkitco/agent-config-templates, CC BY 4.0). Save as .github/instructions/tests.instructions.md (the name must end in .instructions.md) and delete this comment. -->

- Use [Vitest and Testing Library]. Don't use [Enzyme].
- One behavior per test. Name tests "does X when Y".
- Query by role or label, not by test id, unless no accessible name exists.
- No network calls or real clock in unit tests; mock them.
