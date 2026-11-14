# AGENTS.md — PyTorch machine learning project (training, evaluation, inference)

Drop this file at the repo root as `AGENTS.md`. Coding agents (Codex, Cursor, Claude Code via `@AGENTS.md`, Copilot, etc.) read it for project conventions.

**Stack assumption:** a Python [3.11 / 3.12] project using [PyTorch 2.x], managed with [uv / pip + venv / conda], with configs in [YAML via Hydra / OmegaConf / plain dataclasses] and experiment tracking in [Weights & Biases / MLflow / TensorBoard / none]. Replace bracketed defaults if your setup differs.

## Project

- Project: [NAME] — [one line: task, e.g. "image classifier for X" / "fine-tune a small LM for Y"]
- Data: [where it lives, e.g. `data/` (git-ignored) or `s3://[bucket]/[prefix]`], [size], [license / PII notes]
- Primary metric: [e.g. val macro-F1] on [split]; baseline to beat: [value + run id]
- Hardware: [local GPU model / CPU-only / cluster], [single GPU / DDP / FSDP]

## Commands

| Task | Command |
|------|---------|
| Install deps | `[uv sync]` |
| Lint / format | `[uv run ruff check .]` / `[uv run ruff format --check .]` |
| Type check | `[uv run mypy src]` |
| Unit tests (fast, CPU) | `[uv run pytest -q]` |
| Smoke train (tiny, CPU, seconds) | `[uv run python -m [pkg].train trainer.max_steps=20 data.subset=64 device=cpu]` |
| Evaluate a checkpoint | `[uv run python -m [pkg].evaluate ckpt=[path]]` |
| Full training runs, sweeps, cluster jobs, paid GPU time | **never launched by the agent** — a human starts these |

A change is not done until lint, tests and the smoke train pass. "It should work on the GPU" is not evidence; say what you actually ran.

## Layout

- `src/[pkg]/data/` — datasets, transforms, splits; `models/` — `nn.Module`s; `train.py`, `evaluate.py`, `infer.py` — entry points
- `configs/` — every hyperparameter lives here, not hard-coded in Python
- `tests/` — shape, determinism and small end-to-end tests
- `notebooks/` — exploration only; anything reused moves into `src/` with a test
- `data/`, `outputs/`, `checkpoints/`, `wandb/`, `mlruns/` — git-ignored; never commit data, weights or run artifacts

## Data rules (the important part)

- **No leakage.** Splits are fixed and reproducible (saved split files or a seeded, documented rule). Fit normalizers, tokenizers' vocab, scalers and feature selection on the training split only. Group-aware splits when rows share a user, patient, session or document.
- **Never tune on the test set.** Model selection and early stopping use validation only; report test once per final model.
- Don't modify, re-download, or delete raw data in place; derived data is written to a new versioned path.
- Validate inputs at load time (shapes, dtypes, label range, NaNs) and fail loudly.
- Don't print or log raw rows that may contain personal data.

## PyTorch conventions

- Device-agnostic code: pass `device` in, use `.to(device)`; no hard-coded `.cuda()`. Everything must run on CPU for tests.
- Set seeds (`torch`, `numpy`, `random`) from config and log them; document any remaining nondeterminism (e.g. cuDNN, atomics).
- `model.train()` / `model.eval()` set explicitly; evaluation runs under `torch.inference_mode()` (or `no_grad`).
- `optimizer.zero_grad(set_to_none=True)` each step; scale the loss when using gradient accumulation; use `torch.amp` autocast + `GradScaler` only when mixed precision is enabled in config.
- Don't call `.item()`, `.cpu()` or `print` on tensors inside the hot training loop except at the logging interval — it forces GPU syncs.
- Comment tensor shapes at module boundaries (e.g. `# (B, T, D)`); assert shapes in tests, not with silent broadcasting.
- `DataLoader` settings (`num_workers`, `pin_memory`, `persistent_workers`) come from config; transforms in workers must be picklable.
- Checkpoints save model, optimizer, scheduler, scaler, step, config and git SHA; load with `torch.load(..., weights_only=True)` where possible and never load untrusted pickles.
- Distributed code ([DDP / FSDP]): only rank 0 logs and saves; use `DistributedSampler.set_epoch`; metrics are reduced across ranks before reporting.

## Experiments and results

- Every reported number has a run id / config / commit behind it. Never invent, round up or "estimate" metrics; if you didn't run it, say so.
- Compare against the baseline on the same split, metric and seed set; mention variance when the difference is small.
- A changed default hyperparameter, loss, metric or split definition is called out in the PR — it makes old runs incomparable.

## Testing

- Shape tests for each model forward pass on a tiny batch.
- An overfit-one-batch test: loss drops on a fixed tiny batch within N steps.
- Determinism test for data splits and preprocessing.
- Metric functions tested against hand-computed examples.
- Keep tests CPU-only and under [60] seconds total; mark slow/GPU tests and skip them by default.

## Review habits

- Small diffs; one idea per change. Separate refactors from changes that alter results.
- Call out new dependencies (especially ones with compiled extensions or CUDA pins).
- Don't reformat unrelated files, mass-edit notebooks, or commit notebook outputs.

## Ask first (do not do without a human)

- Launching full training, sweeps, cluster or cloud GPU jobs, or anything that costs money
- Changing data splits, the primary metric, or the evaluation protocol
- Deleting or overwriting checkpoints, datasets or experiment runs
- Downloading new datasets or pretrained weights (licenses and terms vary), or uploading data or models anywhere (Hugging Face Hub, buckets, model registries)
- Upgrading PyTorch / CUDA versions

## How to customize

1. Fill `[BRACKETED]` values and delete the parts you don't use (e.g. distributed training, experiment tracking).
2. Keep the file short — agents follow short files better.
3. Optionally add a one-line `CLAUDE.md` with `@AGENTS.md` so Claude Code shares this file.

---

### Disclaimer

Written with AI assistance by [AgentKit Co](https://agentkitco.com/). Starting point, not a guarantee — review against your project, the current PyTorch documentation, your data's license and privacy obligations, and your own testing. PyTorch is a trademark of The Linux Foundation; this file is not affiliated with or endorsed by the PyTorch project.

### Want more?

- Free templates + generator: https://agentkitco.com/tools/agents-md-generator/
- [AI Coding Agent Config Kit](https://agentkitco.com/products/agent-config-kit/) ($14) — 12-stack AGENTS.md set, Cursor rules, Copilot files
- [AI PR Review Kit](https://agentkitco.com/products/ai-pr-review-kit/) ($17) — blockers-first checklists, paste-ready PR comments, review prompts for Claude Code & Cursor

License: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — use and adapt with attribution to AgentKit Co.
