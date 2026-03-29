# AGENT_GUIDE.md

## What This Repo Is

`autoresearch-mlx` is an Apple Silicon / MLX port of Karpathy's `autoresearch` idea: a tightly scoped repo designed for autonomous model-improvement loops.

The core idea is simple:
- one file is the experimental surface: `train.py`
- one metric decides whether a change is good: `val_bpb`
- training runs on a fixed wall-clock budget
- winners are kept in git, losers are reverted

This is **not** a general training framework and **not** a broad codebase for feature work. It is an experiment harness.

## Why This Exists

This repo exists so an AI agent can do bounded, repeatable research on Apple Silicon hardware without constantly renegotiating the rules.

It provides:
- a fixed evaluation harness
- a fixed data/tokenizer pipeline
- a single mutable research surface
- a clear keep-or-revert loop
- a ledger of results in `results.tsv`

The point is not to build the prettiest trainer. The point is to discover changes that actually improve `val_bpb` under the fixed budget on MLX hardware.

## When To Use It

Use this repo when:
- you want an autonomous or semi-autonomous agent to run repeated training experiments
- you want to optimize model/training behavior under a **fixed 5-minute training budget**
- you care about Apple Silicon / MLX-native performance
- you want git-backed experimentation with strict keep/discard discipline
- you want to compare ideas on the **same hardware baseline**, not against random published numbers

## When Not To Use It

Do **not** use this repo for:
- generic ML experimentation across many files
- changing the evaluation rules to make results look better
- dependency churn or framework migration work
- large multi-file refactors "because the code could be cleaner"
- porting PyTorch/CUDA tricks blindly into MLX
- comparing results across different machines as if they were directly equivalent

If your goal is broad framework development, dataset work, packaging, or tooling, this is the wrong repo.

## The Mental Model

Think of the repo like a lab setup:

- `prepare.py` = the lab apparatus
  - data
  - tokenizer
  - dataloader
  - evaluation
  - fixed constraints
- `train.py` = the experiment surface
  - model
  - optimizer
  - architecture
  - hyperparameters
  - training loop
- `results.tsv` = the score ledger
- `program.md` = the operating procedure
- `run.log` = the evidence from the last run

If you change the apparatus, you're cheating the experiment.
If you change `train.py`, you're doing research.

## Core Files and Their Roles

### `README.md`
Human-oriented repo overview. Read this first for context.

### `program.md`
The autonomous experiment protocol. This is the operational contract for long-running agent loops.

### `prepare.py`
**Read-only for normal experimentation.**

It defines:
- cache locations
- data prep
- tokenizer training/loading
- dataloader behavior
- evaluation harness
- fixed experiment constants

Do not modify this during normal research loops unless a human explicitly asks you to change the apparatus itself.

### `train.py`
**This is the only normal edit target.**

This is where you experiment with:
- architecture
- optimizer logic
- hyperparameters
- batch sizing
- learning-rate behavior
- regularization
- other MLX-safe training ideas

### `results.tsv`
Tab-separated experiment ledger.

Record:
- commit
- val_bpb
- peak memory
- keep/discard/crash
- short description

This file is part of the experimental memory of the repo.

## Environment Expectations

This repo expects:
- Apple Silicon Mac
- Python 3.10+
- `uv`
- MLX dependencies from `pyproject.toml`
- prepared cache under `~/.cache/autoresearch/`

Expected prepared artifacts include:
- parquet shards under `~/.cache/autoresearch/data`
- tokenizer artifacts under `~/.cache/autoresearch/tokenizer`

If these do not exist, `uv run prepare.py` must be run before a real experiment.

## Non-Negotiable Rules

1. **Do not modify `prepare.py` during normal experiment loops.**
2. **Do not add dependencies** unless a human explicitly asks.
3. **Do not change the evaluation contract** in order to game the metric.
4. **Edit only `train.py`** for routine research.
5. **Never use blind `git add -A`** in monorepo contexts.
6. **Compare against the local hardware baseline**, not CUDA results or another machine's run.
7. **Prefer simple winning changes** over complicated tiny gains.
8. **Use evidence, not vibes.** A change is good because `val_bpb` improved under the fixed rules.

## Quickstart For Agents

If you are an AI agent dropped into this repo cold, do this in order:

1. Read:
   - `README.md`
   - `AGENT_GUIDE.md`
   - `program.md`
   - `train.py`
2. Verify prepared cache exists at `~/.cache/autoresearch/`
3. If cache is missing, stop and tell the human to run:
   - `uv run prepare.py`
4. Check `results.tsv` to understand the current local history
5. Establish or confirm the baseline on this hardware
6. Make a **small** change to `train.py`
7. Commit only the intended file(s)
8. Run:
   - `uv run train.py > run.log 2>&1`
9. Inspect:
   - `grep "^val_bpb:\|^peak_vram_mb:" run.log`
10. Log the result in `results.tsv`
11. Keep or revert based on the result
12. Repeat

## The Experiment Loop, in Plain English

1. Start from the current kept baseline
2. Try one idea in `train.py`
3. Run one bounded experiment
4. Read the result
5. If it wins, keep it
6. If it loses, revert it
7. Move on to the next idea

This is hill-climbing with discipline, not wandering.

## How To Judge Success

The primary metric is:
- **lower `val_bpb`**

Secondary considerations:
- memory use should stay reasonable
- complexity should not explode for tiny gains
- the run should complete cleanly
- the change should be understandable enough to maintain

A good change:
- improves `val_bpb`
- does not destabilize runs
- does not require ugly hacks unless the gain is clearly worth it

A mediocre change:
- slightly improves the score but adds complexity with no clear upside

A bad change:
- crashes
- silently breaks training semantics
- worsens `val_bpb`
- bloats memory badly
- makes the code much uglier for trivial benefit

## Common Failure Modes

### 1) Comparing against the wrong baseline
Do not compare Apple Silicon results against upstream CUDA results as if they are directly interchangeable.

### 2) Editing the wrong file
If you start changing `prepare.py`, you are changing the lab, not the experiment.

### 3) Porting PyTorch assumptions directly into MLX
Not every optimizer trick, memory assumption, or throughput tradeoff carries over.

### 4) Getting fooled by one lucky run
A single better result is useful, but treat dramatic claims carefully unless the change is robust.

### 5) Adding too much complexity
A tiny gain is often not worth a pile of fragile code.

### 6) Forgetting monorepo-safe git hygiene
In bigger workspaces, stage only the intended `autoresearch-mlx/` files.

## Practical Agent Heuristics

When choosing what to try next, prefer:
- one-variable changes
- throughput-aware ideas
- simplifications that may improve step efficiency
- architecture changes that are plausible on MLX
- optimizer or schedule adjustments that are easy to reason about

Avoid:
- giant rewrites between runs
- changing many interacting variables at once
- speculative complexity with no clear mechanism
- anything that blurs cause and effect in the results

## Suggested Commit Discipline

Good experiment commit messages look like:
- `experiment: lower matrix lr`
- `experiment: reduce depth to 4`
- `experiment: change mlp activation to silu`

They should describe the idea plainly, not theatrically.

## What To Tell a Human Quickly

If a human asks what this repo does, a good short answer is:

> This is an MLX-native Apple Silicon port of autoresearch: a repo where an agent repeatedly edits `train.py`, runs a fixed-budget training experiment, measures `val_bpb`, keeps winning changes, and reverts losing ones.

If they ask why it matters:

> It turns model tuning into a disciplined autonomous search loop instead of ad hoc experimentation.

## Bottom Line

This repo is a **bounded autonomous research harness**.

Respect the boundaries:
- fixed apparatus
- one mutable surface
- one metric
- one loop

If you follow that, this repo is powerful.
If you ignore that, it turns into meaningless training theater fast.
