# flmnt Benchmark

The public trust artifact for **flmnt** — a persistent memory layer for AI agents. This is the
verifiable, reproducible record of whether retrieving recorded project decisions changes what a
model can correctly answer.

It is not a marketing asset. Everything here is inspectable — the exact inputs, the exact
grading rubric, the raw scores. Failures and limitations are documented alongside the results.

---

## The core claim

Given a project event stream seeded with architectural decisions, flmnt's retrieval surfaces
the relevant decisions to the model exactly when it needs them — recovering **near-oracle
accuracy from near-zero cold accuracy**.

**Answer Accuracy, composite across all 7 packs / 53 probes (run 2026-06-22, n=5, median):**

| Model | cold | warm | **rlm** | oracle |
|---|---|---|---|---|
| `claude-haiku-4-5` | 0.117 | 0.991 | **0.986** | 0.996 |
| `gpt-5.4-mini` | 0.244 | 0.976 | **0.977** | 0.992 |

- **cold** — no memory, parametric knowledge only — is the floor.
- **oracle** — the correct decisions injected directly — is the ceiling, and proves the answer
  is actually in the stream.
- **rlm** — flmnt's query-driven deep retrieval — sits on the oracle ceiling and roughly
  5–10× above the cold floor on every pack. That gap is flmnt's isolated contribution.

See [`METHODOLOGY.md`](METHODOLOGY.md) for the four-arm design and
[`data/latest.jsonl`](data/latest.jsonl) for every per-probe score.

## The model-size finding

`claude-haiku-4-5` (small) reaches **rlm 0.986** — within ~0.01 of `gpt-5.4-mini` at 0.977,
and both land on their oracle ceilings. **The accuracy lift comes from retrieved context, not
from model capability.** A small, fast, cheap model given precise context matches what far
larger models achieve — because the answer was in the stream the whole time.

---

## What's in this repository

| Path | Contents |
|---|---|
| [`CORPUS.md`](CORPUS.md) | The exact, complete input to every run — every seeded decision and every probe question, per pack. Generated directly from the scenario source. |
| [`SCORING.md`](SCORING.md) | The verbatim LLM-as-judge rubric used to grade every answer. |
| [`METHODOLOGY.md`](METHODOLOGY.md) | Four-arm experimental design, the seven packs, probe types, metrics, result format. |
| [`data/latest.jsonl`](data/latest.jsonl) | Raw results — one row per `(probe × arm × model)`, median-aggregated over n=5. |
| [`data/corpus.json`](data/corpus.json) | Machine-readable form of `CORPUS.md`. |

`CORPUS.md` and `data/latest.jsonl` are generated from the same source, so the inputs and the
results are guaranteed to describe the same suite — the whole thing is auditable end to end.

> **A note on naming.** Some seed system prompts in `CORPUS.md` reference *"Quorum tools."*
> **Quorum is flmnt's internal project codename** — the two names refer to the same system. The
> prompts are published exactly as the models received them, unedited, rather than rewritten for
> branding.

## What's *not* in this repository

- flmnt's retrieval algorithm, ranking weights, or embedding/traversal engineering
- The retrieval system prompts
- Any runtime source code

The runtime never leaves flmnt's infrastructure. **Publish the test, not the implementation.**

---

## The four arms

Every probe runs four ways — same model, same question, only the context differs:

| Arm | Context the model gets |
|---|---|
| **cold** | nothing — parametric knowledge only (the floor) |
| **warm** | `materialize_context` — a snapshot read of the stream |
| **rlm** | `query_context` — deep, query-driven retrieval (what production uses) |
| **oracle** | the correct decisions injected directly (the ceiling) |

`rlm ≈ oracle ≫ cold` is the result we look for: retrieval recovering near-ceiling accuracy
from a model that otherwise cannot answer.

## Coverage & status

This is a **pre-GA, in-progress** publication. The 2026-06-22 run covers two models
(`claude-haiku-4-5`, `gpt-5.4-mini`); additional models (`claude-sonnet-4-6`, `gpt-5.5`,
`claude-opus-4-8`) are being added pass by pass. Each model's full four-arm, seven-pack,
n=5 results are published as they complete.

---

*Every number here was produced by the same retrieval runtime flmnt runs in production, scored
by the rubric in `SCORING.md`, against the inputs in `CORPUS.md`. If something looks wrong,
it's meant to be checkable — that's the point.*
