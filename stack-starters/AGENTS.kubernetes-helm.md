# AGENTS.md — Kubernetes manifests + Helm charts

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** Kubernetes [1.29+] manifests and Helm [3.x] charts in Git, optionally overlaid with [Kustomize], validated with [`kubeconform`] and [`helm lint`], policy-checked with [Kyverno / OPA Gatekeeper / Conftest], and delivered by [Argo CD / Flux / a CI pipeline] — not by hand. Replace bracketed defaults if your setup differs, and check the current Kubernetes and Helm docs for your versions, since APIs are deprecated and removed between releases.

## Project

- Repo: [PROJECT NAME] — [which services or platform components it deploys]
- Clusters / environments: [dev, staging, prod] · how they map to folders or values files: [`envs/<env>/` / `values-<env>.yaml`]
- Delivery: [GitOps — Argo CD syncs `main`] · the agent never deploys directly
- Secrets: [External Secrets / Sealed Secrets / SOPS / Vault] — plain secret values never live in this repo

## Commands (local, read-only against clusters)

| Task | Command |
|------|---------|
| Lint a chart | `helm lint charts/[chart] -f charts/[chart]/values-[env].yaml` |
| Render a chart | `helm template [release] charts/[chart] -f charts/[chart]/values-[env].yaml > /tmp/rendered.yaml` |
| Render an overlay | `kubectl kustomize envs/[env]` |
| Schema-validate rendered YAML | `kubeconform -strict -summary -kubernetes-version [1.29.0] /tmp/rendered.yaml` |
| Policy check | `[conftest test /tmp/rendered.yaml]` / `[kyverno apply policies/ --resource /tmp/rendered.yaml]` |
| Diff against the live cluster (read-only) | `[kubectl diff -f /tmp/rendered.yaml]` / `[helm diff upgrade ...]` — only if you have read access to that context |
| Chart unit tests | `[helm unittest charts/[chart]]` |
| Apply / install / upgrade | **never run by the agent** — GitOps or CI applies |

A change is not done until every touched chart lints, renders for each environment it ships to, and the rendered YAML passes schema and policy checks. If you can't reach a cluster, say so plainly and rely on render + validate; don't guess at live state.

## Layout

- `charts/<chart>/` — `Chart.yaml`, `values.yaml` (safe defaults), `values-<env>.yaml`, `templates/`, `templates/_helpers.tpl`, `tests/`
- `base/` + `envs/<env>/` — Kustomize base and overlays, if used
- `policies/` — admission/policy rules the CI runs against rendered output
- `apps/` — [Argo CD Applications / Flux Kustomizations], one per service and environment
- Never commit `charts/*/charts/*.tgz` unless the repo vendors dependencies on purpose, kubeconfig files, or decrypted secrets.

## Conventions

1. **Render, then reason.** Before and after a change, render the affected chart or overlay for each environment and compare. Explain the rendered diff in the PR, not just the template diff.
2. **Never put secrets in Git.** No values in `Secret` manifests, `values*.yaml`, `ConfigMap`s or env vars. Reference the existing secret mechanism ([ExternalSecret / SealedSecret / SOPS file]).
3. **Requests and limits on every container.** Set CPU and memory requests; set a memory limit. Follow the house rule on CPU limits ([set / don't set]). Don't lower another service's resources to make room.
4. **Probes that mean something.** `readinessProbe` gates traffic, `livenessProbe` only restarts a truly stuck process, and `startupProbe` covers slow starts. Don't point liveness at a dependency check.
5. **Secure by default.** `runAsNonRoot: true`, `allowPrivilegeEscalation: false`, `readOnlyRootFilesystem: true` where the app allows, drop all capabilities, no `privileged`, `hostNetwork`, `hostPath` or `hostPID` unless the ask is specifically about them.
6. **Pin images by tag and digest** ([`image@sha256:...`] or an immutable tag). Never `latest`. Image bumps change only the image field.
7. **Labels and selectors.** Use the standard `app.kubernetes.io/*` labels from `_helpers.tpl`. Never change a Deployment's or StatefulSet's `spec.selector` — it's immutable and forces a delete and recreate.
8. **Availability.** Keep `replicas`, `PodDisruptionBudget`, `topologySpreadConstraints` and rollout strategy (`maxUnavailable`/`maxSurge`) as they are unless asked. Changes that reduce redundancy in prod get called out.
9. **Templates stay readable.** Put repeated logic in `_helpers.tpl`, use `toYaml | nindent` for blocks, `required` for values with no safe default, and `quote` for strings. Add any new value to `values.yaml` with a comment and a safe default, and to `values.schema.json` if it exists.
10. **Current APIs only.** Use non-deprecated `apiVersion`s for the target cluster version. If you find a deprecated one, mention it rather than silently migrating it in an unrelated change.
11. **Network and RBAC are least privilege.** New `NetworkPolicy`, `Role` or `ClusterRole` rules list the specific resources and verbs needed — no `*` verbs or resources. Prefer namespaced `Role` over `ClusterRole`.
12. **Stateful things are fragile.** `StatefulSet` volume claim templates, `PersistentVolumeClaim` size or storage class, and anything with a `PersistentVolume` are ask-first; some fields can't be changed in place.

## Testing

- `helm lint` and `helm template` succeed for every environment values file the chart ships with.
- `kubeconform -strict` passes against the target Kubernetes version; policy checks pass.
- If chart unit tests exist, add or update one for new template logic (conditionals, loops, `required` values).
- For a change to a running service, list in the PR what will roll (which Deployments restart) and why.

## Review habits (before you say "done")

- [ ] Diff is scoped to the ask — no drive-by reformatting of YAML or values files
- [ ] Rendered diff checked for every affected environment and summarized in the PR
- [ ] No secrets, kubeconfigs or decrypted files committed
- [ ] Requests/limits, probes and securityContext present on new or changed containers
- [ ] No `latest` tags; no change to immutable selectors; no deprecated APIs introduced
- [ ] RBAC and NetworkPolicy changes are least privilege; anything that widens access is called out
- [ ] Lint, render, schema and policy checks pass
- [ ] If you opened a PR: blockers-first summary (what breaks if wrong → what changed → how to verify)

## Boundaries — ask first

- Any `kubectl apply`, `delete`, `edit`, `patch`, `scale`, `rollout restart`, `helm install/upgrade/uninstall/rollback` or `argocd app sync` against a real cluster
- Switching kube contexts or touching prod kubeconfigs and credentials
- Changes to CRDs, admission webhooks, cluster-wide RBAC, namespaces, `ResourceQuota`, `LimitRange` or cluster add-ons (ingress controller, cert-manager, CNI)
- Deleting resources, changing `StatefulSet` storage, PVCs or reclaim policies
- Reducing replicas or PDBs in prod, or exposing a new public `Ingress` / `LoadBalancer`
- Upgrading chart dependencies, the Kubernetes target version or Helm itself

## How to customize

1. Fill `[BRACKETED]` values and delete options you don't use (Kustomize vs Helm, policy engine, delivery tool).
2. Delete sections that do not apply (keep the file short — agents follow short files better).
3. Optionally add a one-line `CLAUDE.md` with `@AGENTS.md` so Claude Code shares this file.

---

### Disclaimer

Written with AI assistance by [AgentKit Co](https://agentkitco.com/). Starting point, not a guarantee — review against your project and keep real enforcement (CI validation, admission policies, protected clusters, GitOps review) in place.

### Want more?

- Free templates + generator: https://agentkitco.com/tools/agents-md-generator/
- [AI Coding Agent Config Kit](https://agentkitco.com/products/agent-config-kit/) ($14) — 12-stack AGENTS.md set, Cursor rules, Copilot files
- [AI PR Review Kit](https://agentkitco.com/products/ai-pr-review-kit/) ($17) — blockers-first checklists, paste-ready PR comments, review prompts for Claude Code & Cursor

License: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — use and adapt with attribution to AgentKit Co.
