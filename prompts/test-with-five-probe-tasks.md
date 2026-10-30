# Test the file with five probe tasks

**When to use:** You changed your instruction file and want to know if it works.

Copy the block below into your coding agent. Replace the [BRACKETED] parts.

```text
Pretend you are a new agent that has only read [INSTRUCTION FILE]. Do not look at the code yet.

Write five small probe tasks that would reveal whether the file is good enough, for example: "add a new endpoint", "fix a failing test", "rename a module", "add a dependency", "update the docs". For each task, say (1) which lines of the file you would rely on, (2) which questions the file leaves unanswered, and (3) which rule you would be most tempted to ignore, and why.

Finish with the three edits that would improve the file most.
```

---

Free prompt from [AgentKit Co](https://agentkitco.com) (CC BY 4.0). Want more? The [AI Coding Agent Config Kit](https://agentkitco.com/products/agent-config-kit/) ($14) has 12 audit and test prompts, plus AGENTS.md templates for 12 stacks and ready Cursor, Claude Code and Copilot rule files.
