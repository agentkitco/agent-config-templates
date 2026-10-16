# Draft AGENTS.md from the repo

**When to use:** You have no instruction file yet and want an agent to propose one.

Copy the block below into your coding agent. Replace the [BRACKETED] parts.

```text
You are setting up an AGENTS.md for this repository. Do not write the file yet.

Step 1: Explore. Read the README, dependency manifests, CI config, lint, format and test config, and the top two levels of the directory tree.
Step 2: List the commands you found for: install, run, build, test all, test one file, lint, format, type-check. For each, say where you found it. If you cannot find one, write "not found". Do not guess commands.
Step 3: List conventions that are visible in the code and differ from the defaults of [LANGUAGE / FRAMEWORK]. Give one file path as evidence for each.
Step 4: List hazards: generated files, secrets, migrations, vendored code, slow or destructive commands.
Step 5: Ask me up to 5 questions you cannot answer from the repository.

Stop and wait for my answers. After that, draft AGENTS.md under [120] lines with these sections: Project, Commands, Layout, Conventions, Boundaries, Git. Leave out anything a developer would learn by reading the code or that [TOOL] already knows.
```
