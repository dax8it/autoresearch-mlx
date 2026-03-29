# HERMES_ONBOARDING.md

This file is for Hermes or any similarly capable coding/research agent that is being asked to operate inside `autoresearch-mlx`.

Read this **after** `README.md` and `AGENT_GUIDE.md`.

## Your Job

Your job is not to "improve the repo" in some vague sense.

Your job is to run a disciplined experimental search over `train.py` and find changes that improve `val_bpb` on this machine under the fixed MLX training budget.

You are operating a research loop, not a product roadmap.

## The One-Sentence Mental Model

You are an autonomous researcher working inside a fixed lab setup:
- `prepare.py` defines the apparatus
- `train.py` is the experiment surface
- `results.tsv` is the score ledger
- git is the keep-or-revert memory

## Read These Files First

1. `README.md`
2. `AGENT_GUIDE.md`
3. `program.md`
4. `train.py`
5. `results.tsv`

Do not start experimenting before reading them.

## What You Are Allowed To Change

Normal experimentation:
- `train.py`
- `results.tsv`
- `run.log` as runtime output

Only change anything else if a human explicitly asks.

## What You Must Not Change

Do not modify during ordinary experiment loops:
- `prepare.py`
- `pyproject.toml`
- `uv.lock`
- evaluation logic in the data/eval harness
- repo structure

Do not install new packages unless explicitly asked by a human.

## Success Metric

Primary metric:
- **lowest `val_bpb` wins**

Secondary judgment:
- memory should remain reasonable
- code complexity should stay justified
- changes should actually run and finish

## The Correct Workflow

### 1) Check environment
Verify:
- this is an Apple Silicon / MLX environment
- `~/.cache/autoresearch/` exists
- tokenizer + parquet shards exist

If the cache is missing, stop and tell the human to run:

```bash
uv run prepare.py
```

### 2) Understand the current baseline
Read `results.tsv`.
Do not assume a result from another machine is the right baseline here.
Hardware matters.

### 3) Try one idea at a time
Prefer small, interpretable changes.
Do not combine five ideas into one run unless there is a very good reason.

### 4) Commit the experiment
Use clear experiment commit messages.
Stage only the intended files.

In monorepo contexts, never use blind `git add -A`.

### 5) Run the experiment cleanly
Use:

```bash
uv run train.py > run.log 2>&1
```

Do not flood your context with full live output unless a human explicitly wants that.

### 6) Read the result
Extract:
- `val_bpb`
- `peak_vram_mb`
- crash behavior if any

### 7) Log the run
Append the result to `results.tsv`.
Use `keep`, `discard`, or `crash` honestly.

### 8) Decide cleanly
- If it improved: keep it
- If it did not: revert it cleanly
- If it crashed: record it and move on

## How To Think

Good ideas in this repo usually come from:
- better step efficiency
- better optimization under fixed wall-clock time
- simpler/faster architectures that win by fitting more useful steps into the budget
- MLX-aware tradeoffs, not CUDA instincts copied blindly

Bad instincts in this repo:
- treating bigger as automatically better
- importing upstream PyTorch/CUDA habits without translation
- optimizing for aesthetics instead of score
- changing too many variables at once
- rewriting the harness when the experiment is the thing that should change

## Simplicity Rule

Simplicity matters.

A tiny gain with ugly complexity may not be worth keeping.
A comparable result with simpler code is often a win.
A deletion that improves results is especially valuable.

Do not worship complexity.

## Failure Handling

If a run crashes:
1. inspect `run.log`
2. determine whether it was a trivial bug or a bad idea
3. fix only if the fix is obvious and worthwhile
4. otherwise mark it as `crash`, revert, and move on

Do not get stuck polishing a dead idea forever.

## What Not To Confuse

Do not confuse:
- repository improvement with experiment success
- elegance with empirical improvement
- one lucky run with a robust idea
- upstream parity with MLX fitness

This fork is intentionally different from upstream.
Respect that.

## Communication Style

When reporting back to a human, be concise and concrete:
- what changed
- what the result was
- whether it was kept or discarded
- what you want to try next

No hype. No hand-waving.

## Good Short Summary To Keep In Mind

> This repo exists to let an agent repeatedly modify `train.py`, run a fixed-budget MLX training experiment, measure `val_bpb`, and keep only changes that actually win.

If you keep that sentence in your head, you will use this repo correctly.
