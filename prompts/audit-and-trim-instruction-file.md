# Audit and trim an instruction file

**When to use:** Your AGENTS.md or CLAUDE.md has grown and you suspect half of it is noise.

Copy the block below into your coding agent. Replace the [BRACKETED] parts.

```text
Audit the file [PATH TO INSTRUCTION FILE]. Do not edit it yet.

Sort every line into one of five groups:
KEEP: concrete, checkable, and not obvious from the code.
VERIFY: refers to a command, path or tool. Check that it exists and report what you found.
CUT: vague ("write clean code"), generic advice any agent already follows, or a copy of README content.
MERGE: duplicates or overlaps another line.
MOVE: only matters for part of the repo, so it belongs in a path-scoped rule or a nested file.

Output a table: line number | group | reason. Then show the trimmed file and a one-line summary of how many lines you removed. Wait for my approval before writing anything.
```
