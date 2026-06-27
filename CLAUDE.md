# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

VinUni AICB Day 22 — Track 3 DPO/ORPO Alignment lab. An educational pipeline that builds an SFT checkpoint, prepares preference data, runs DPO training, and evaluates the result. Python 3.10–3.12.

## Notebooks: edit the `.py`, never the `.ipynb`

Notebook sources live in `notebooks/*.py` (jupytext `py:percent` format). The `.ipynb` files are **generated** from them via `jupytext --to notebook --update` and are git-ignored.

- **Always edit `notebooks/NN_*.py`** — edits to `.ipynb` will be overwritten on the next sync.
- `colab/*.ipynb` are the exception: standalone Colab notebooks, edited directly.
- Core pipeline is **NB1–NB4** (`01`–`04`); **NB5 (GGUF) and NB6 (benchmark) are optional/bonus**.

## Commands

Everything is wrapped in the `Makefile` (auto-detects Colab vs laptop venv). Key targets:

- `make setup` — install deps + smoke check (runs `setup-colab.sh` or `setup-laptop.sh`)
- `make smoke` — pre-training check: imports + GPU visible (`python scripts/verify.py --smoke`)
- `make test` — `pytest -q scripts/`; **CPU-only smoke tests, no GPU/training** — this is the gate that runs locally
- `make pipeline` — run core notebooks NB1→NB4 (~30 min on T4)
- `make verify` — pre-submission gatekeeper (checks artifacts + REFLECTION.md edited)
- Single test: `pytest -q scripts/test_smoke.py -k <name>`

## Compute tiers

Behavior is driven by `COMPUTE_TIER` in `.env` (copy from `.env.example`):
- `T4` (default) — Qwen2.5-3B-4bit, 2k pref pairs, max_length=512
- `BIGGPU` — Qwen2.5-7B-4bit, 5k pref pairs, max_length=1024 (needs `requirements-biggpu.txt` extras)

Full DPO/SFT training **requires a CUDA GPU**. On CPU/Mac only `make test` (and code edits) work; `make smoke` will fail by design. Use the Colab T4 path (see `HARDWARE-GUIDE.md`).

## Gotchas

- **TRL trainers**: pass `processing_class=tokenizer`, **never** `tokenizer=tokenizer`. The `tokenizer=` arg was removed in trl ≥ 0.13 and a fresh install resolves to 0.19.x. `scripts/test_smoke.py` enforces this — keep it passing.
- The Unsloth version pin locks the Unsloth + TRL compatibility matrix; bump `unsloth`, `transformers`, and `trl` together.

## Style

- ruff: `line-length = 100`, target `py310` (config in `pyproject.toml`).
- Comments in Vietnamese are fine and common; keep identifiers in English.
