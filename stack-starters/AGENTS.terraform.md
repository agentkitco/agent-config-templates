# AGENTS.md — Terraform (infrastructure as code)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** Terraform [1.6+] (or OpenTofu [1.6+]) managing [AWS / Azure / GCP / Cloudflare / other] with remote state in [S3 + DynamoDB lock / Terraform Cloud / GCS / azurerm backend], providers pinned in `.terraform.lock.hcl`, linting with [tflint], security scanning with [checkov / trivy config / tfsec], and plans reviewed in CI with [GitHub Actions / Atlantis / Terraform Cloud runs]. Replace bracketed defaults if your project differs, and check the current Terraform and provider docs for your versions, since arguments and defaults change between releases.

## Project

- Repo: [PROJECT NAME] — [what infrastructure it manages]
- Environments: [dev / staging / prod] in [`envs/<env>/` folders / workspaces / separate accounts]
- State backend: [backend + bucket/org name] · state locking: [yes, via ...]
- Cloud accounts/projects: [IDs or names, which env maps to which]

## Commands (run inside the root module or env folder)

| Task | Command |
|------|---------|
| Format | `terraform fmt -recursive` (CI uses `terraform fmt -check -recursive`) |
| Init (no backend, for local checks) | `terraform init -backend=false` |
| Validate | `terraform validate` |
| Lint | `[tflint --recursive]` |
| Security scan | `[checkov -d .]` / `[trivy config .]` |
| Tests | `[terraform test]` (`*.tftest.hcl`) |
| Plan | `terraform plan -out=tfplan` (needs credentials; read-only against real state) |
| Apply | **never run by the agent** — humans or CI apply reviewed plans |

A change is not done until `fmt -check`, `validate`, lint and the security scan pass. If you have read-only credentials, run `plan` and paste the summary (adds / changes / destroys) in the PR; if you cannot run it, say so plainly.

## Layout

- `modules/<name>/` — reusable modules: `main.tf`, `variables.tf`, `outputs.tf`, `versions.tf`, `README.md`
- `envs/<env>/` — root modules that call modules with env-specific values (`[terraform.tfvars]`)
- `versions.tf` — `required_version` and `required_providers` with version constraints
- `.terraform.lock.hcl` — provider lock file; commit it, update it only on purpose
- Never edit or commit `.terraform/`, `*.tfstate`, `*.tfstate.backup`, `tfplan` files or crash logs.

## Conventions

1. **Follow the existing structure.** Match current module layout, naming and env split. Don't introduce Terragrunt, CDKTF, a new backend or a new wrapper script.
2. **Plan before you claim.** Every change should be explainable as a plan diff. Call out any `destroy` or `-/+` (replace) line explicitly; a replace on a database, bucket, disk or DNS zone is a blocker until a human agrees.
3. **Never touch state directly.** No `terraform state rm/mv`, `import`, `taint`, `force-unlock` or hand-edited state. If refactoring needs it, use `moved {}` / `import {}` blocks in code and say so in the PR.
4. **Protect stateful resources.** Keep `prevent_destroy`, deletion protection, backups, versioning and `skip_final_snapshot = false` where they exist. Don't remove them to make a plan pass.
5. **Pin versions.** Providers use `~>` constraints in `required_providers`; modules from a registry or git use a pinned version or ref, never `main`.
6. **Variables and outputs.** Every variable has a `type` and `description`; add `validation` blocks for values with a known shape. Mark secrets `sensitive = true`. Outputs have descriptions and never expose secrets.
7. **No secrets in code.** No keys, passwords or tokens in `.tf`, `.tfvars` or examples. Read them from [Secrets Manager / Key Vault / Secret Manager / CI env vars] via data sources or variables. Remember sensitive values still land in state.
8. **Least privilege.** IAM policies list specific actions and resources; no `*:*`, no `0.0.0.0/0` ingress on admin ports, no public buckets unless the ask says so and the PR explains why.
9. **Tags and naming.** Apply the standard tags (`[default_tags in the provider]`: owner, env, cost center) and the existing naming pattern so resources can be found and billed.
10. **Prefer `for_each` over `count`** for collections keyed by name, so removing one item does not shift and recreate the rest.
11. **Keep modules small and boring.** No provider blocks inside reusable modules (pass providers in), no hard-coded account IDs or regions, minimal `depends_on`.
12. **Data, not guesses.** Look up AMIs, zones and existing resources with data sources instead of hard-coding IDs copied from the console.

## Testing

- Run `terraform validate` and lint for every touched root module and module.
- Add or update `terraform test` cases (plan-mode `run` blocks with `assert`) when you change module logic, variables or validation.
- For risky changes, include the relevant plan excerpt in the PR rather than a summary from memory.

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by renames (renames recreate resources unless you add `moved {}`)
- [ ] `fmt -check`, `validate`, lint and security scan pass; plan run or noted as not run
- [ ] Plan has no unexpected destroys or replaces; any that exist are called out at the top of the PR
- [ ] No state commands, no `-target` shortcuts, no edited lock file unless the ask was a provider upgrade
- [ ] New variables typed and described; secrets marked `sensitive`; no secrets in code or outputs
- [ ] IAM and network rules are least-privilege; nothing newly public
- [ ] Deletion protection, `prevent_destroy` and backups unchanged
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify)

## Boundaries — ask first

- Running `apply`, `destroy`, or any command that changes real infrastructure or state
- Anything in production, IAM/org policies, networking (VPCs, peering, DNS) or KMS keys
- Changing the backend, state layout, workspaces or moving resources between states
- Upgrading Terraform, providers or pinned module versions
- Deleting or replacing any stateful resource (databases, buckets, volumes, queues)

## How to customize

1. Fill `[BRACKETED]` values and delete options you don't use (cloud, backend, scanner, CI runner).
2. Delete sections that do not apply (keep the file short — agents follow short files better).
3. Optionally add a one-line `CLAUDE.md` with `@AGENTS.md` so Claude Code shares this file.

---

### Disclaimer

Written with AI assistance by [AgentKit Co](https://agentkitco.com/). Starting point, not a guarantee — review against your repo and keep real enforcement (plan review, policy checks, protected applies) in CI.

### Want more?

- Free templates + generator: https://agentkitco.com/tools/agents-md-generator/
- [AI Coding Agent Config Kit](https://agentkitco.com/products/agent-config-kit/) ($14) — 12-stack AGENTS.md set, Cursor rules, Copilot files
- [AI PR Review Kit](https://agentkitco.com/products/ai-pr-review-kit/) ($17) — blockers-first checklists, paste-ready PR comments, review prompts for Claude Code & Cursor

License: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — use and adapt with attribution to AgentKit Co.
