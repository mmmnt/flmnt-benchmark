# flmnt Benchmark

Benchmark suite for **flmnt**, a persistent memory layer for AI agents. It measures whether
retrieving recorded project decisions changes what a model answers correctly. This repository
contains the exact inputs, the grading rubric, and the raw per-probe scores for every run.

---

## Arms

Every probe runs once under each arm with the same model and the same question; only the
retrieved context differs.

| Arm | Context the model receives |
|---|---|
| **cold** | none — parametric knowledge only |
| **warm** | `materialize_context` — a snapshot read of the stream |
| **flat** | vector-similarity retrieval over the stream, no graph traversal |
| **rlm** | `query_context` — query-driven retrieval (the production path) |
| **oracle** | the relevant decisions injected directly |

## Latest run — 2026-06-25

Models: `claude-haiku-4-5`, `claude-opus-4-8`. Judge: `claude-haiku-4-5`. 8 packs, 5 arms,
n=5. Median Answer Accuracy below; scores are identical across both models except where noted.

| Pack | Scenario | cold | warm | flat | rlm | oracle |
|---|---|---|---|---|---|---|
| 1 | Core context recovery | 0.00 | 0.00 | 1.00 | 1.00 | 1.00 |
| 2 | Decision supersession (recency) | 0.00 | 0.00 | 1.00 | 1.00 | 1.00 |
| 3 | Causal traversal | 0.00 | 0.00 | 1.00 | 1.00 | 1.00 |
| 4 | Multi-session handoff | 0.00 | 1.00 | 1.00 | 1.00 | 1.00 |
| 5 | Project isolation | 0.00 | 1.00 | 1.00 | 1.00 | 1.00 |
| 6 | Abstention & hallucination guard | 0.00 | 1.00 | 1.00 | 1.00 | 1.00 |
| 7 | Multi-hop causal chain | 0.00 | 0.00 | 0.95–1.00 | 0.95 | 0.95 |
| 8 | Structural supersession | 0.00 | 0.00 | 0.00 | 1.00 | 1.00 |

Pack 7 flat: 1.00 on `claude-opus-4-8`, 0.95 on `claude-haiku-4-5`.

Per-run history is in [`CHANGELOG.md`](CHANGELOG.md), including the cost/consumption table for
the pack 1 corpus-size sweep. Every per-probe score is in
[`data/latest.jsonl`](data/latest.jsonl). The prior run is preserved in
[`data/2026-06-22-0.jsonl`](data/2026-06-22-0.jsonl).

---

## What's in this repository

| Path | Contents |
|---|---|
| [`CORPUS.md`](CORPUS.md) | The exact, complete input to every run — every seeded decision and every probe question, per pack. Generated directly from the scenario source. |
| [`SCORING.md`](SCORING.md) | The verbatim LLM-as-judge rubric used to grade every answer. |
| [`METHODOLOGY.md`](METHODOLOGY.md) | Experimental design, the packs, probe types, metrics, result format. |
| [`CHANGELOG.md`](CHANGELOG.md) | Per-run results and the fixes applied before each run. No run is deleted. |
| [`data/latest.jsonl`](data/latest.jsonl) | Raw results — one row per `(probe × arm × model)`, n=5. |
| [`data/corpus.json`](data/corpus.json) | Machine-readable form of `CORPUS.md`. |

`CORPUS.md` and `data/latest.jsonl` are generated from the same source, so the inputs and the
results describe the same suite.

> **A note on naming.** Some seed system prompts in `CORPUS.md` reference *"Quorum tools."*
> Quorum is flmnt's internal project codename — the two names refer to the same system. The
> prompts are published as the models received them, unedited, rather than rewritten for
> branding. Where a prompt's verbatim wording would describe retrieval-handling internals, the
> public corpus shows a capability-level summary in its place and marks it as such.

## What's not in this repository

- flmnt's retrieval algorithm, ranking weights, or embedding/traversal engineering
- The retrieval system prompts
- Any runtime source code

The runtime never leaves flmnt's infrastructure; only the test is published.

## Coverage

The 2026-06-25 run covers `claude-haiku-4-5` and `claude-opus-4-8` across 8 packs and 5 arms at
n=5. Additional models are added pass by pass; each model's full results are published as they
complete.
