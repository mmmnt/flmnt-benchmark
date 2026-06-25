# Methodology

This document defines exactly how the flmnt benchmark is built, run, and scored — so every
number in [`data/latest.jsonl`](data/latest.jsonl) can be understood and audited independently.
The complete inputs (every seeded decision and every probe question) are in
[`CORPUS.md`](CORPUS.md); the verbatim grading rubric is in [`SCORING.md`](SCORING.md).

## 1. What the benchmark measures

flmnt is a persistent memory layer for AI agents — recorded project decisions, retrieved on
demand. The benchmark answers one question: **does retrieving those decisions change what a
model can correctly answer?**

Each pack seeds a corpus of recorded decisions into an event stream, then asks the model a set
of probe questions. Each probe compares the model answering *with* retrieval against the same
model answering *without* it, and against a full-context ceiling.

## 2. Experimental design — five arms

Every probe runs across five independent arms. The same model, the same question — only the
context differs:

| Arm | Setup | What it isolates |
|---|---|---|
| **cold** | No tools, no memory. The model answers from parametric knowledge only. | What the model knows with **no** project context — the floor. |
| **warm** | `materialize_context` — a recency/snapshot read of the stream. | flmnt's **snapshot** retrieval tier. |
| **flat** | Vector-similarity retrieval over the stream, no graph traversal. | A single-store **vector-only baseline** — the comparator for graph-based retrieval. |
| **rlm** | `query_context` — query-driven deep retrieval (embedding-anchored causal traversal), escalating to a graph-neighbourhood walk when the direct hit is insufficient. | flmnt's **deep-retrieval** path — what production uses. |
| **oracle** | The correct decisions injected directly into the prompt. | The **ceiling** — the answer is in the stream and the model can use it when handed over directly. |

`cold` is the floor and `oracle` is the ceiling; `warm`, `flat`, and `rlm` sit between them.
The two controls bound the result: without `oracle`, a high retrieval score could mean the
answer was never in the stream; without `cold`, it could mean the model already knew it.
`warm`, `flat`, and `rlm` differ only in retrieval strategy over the same stream — snapshot
read, vector-only similarity, and graph-based query-driven retrieval respectively.

## 3. The eight packs

Each pack targets a distinct memory capability. Exact seeds and probes are in
[`CORPUS.md`](CORPUS.md).

| Pack | Capability under test | Probes |
|---|---|---|
| **1 — Core** | foundational decision recall + premise-rejection + abstention | 15 |
| **2 — Supersession** | decisions replaced over time — report the current one, recognize the superseded one | 5 |
| **3 — Causal Graph** | multi-hop reasoning: problem → decision → outcome | 8 |
| **4 — Multi-Session** | memory persisting across sessions | 6 |
| **5 — Isolation** | project boundary — no cross-contamination between streams | 8 |
| **6 — Hallucination Guard** | resisting fabrication (no recall probes — entirely "don't make it up") | 6 |
| **7 — Causal Chain** | deep causal-chain reasoning | 5 |
| **8 — Structural Supersession** | a decision superseded with no textual staleness marker in the stale entry — identify the current decision via the recorded supersession relationship | 3 |

Packs 2 and 8 both involve superseded decisions but differ in the staleness signal: in **pack
2** the current decision is identifiable from the stream's content and recency; in **pack 8**
the stale entry carries no textual marker, so the only signal that it has been replaced is the
recorded supersession relationship.

## 4. Probe types

| Type | The model must… |
|---|---|
| **recall** | retrieve a recorded decision and its rationale |
| **contradiction** | reject a false premise stated in the question and correct it |
| **abstention** | admit the information is not recorded — not fabricate it |
| **fama** | recognize a decision was superseded, not cite the stale one as current |

## 5. Metrics & scoring

Every answer is graded by an LLM judge (`claude-haiku-4-5`) against the probe's expected
answer and rationale, using the per-type rubric in [`SCORING.md`](SCORING.md) (reproduced
verbatim). Per probe:

| Metric | Source | Question |
|---|---|---|
| **AA** — Answer Accuracy | LLM judge | Does the answer match the expected answer? *(primary metric)* |
| **RF** — Rationale Fidelity | LLM judge | Does the stated reasoning match the recorded rationale? |
| **CR** — Composite | derived | `(AA + RF) / 2` |
| **sub_em** | deterministic | Substring exact-match — a judge-independent secondary signal |

Each probe is run **n = 5** times per arm per model; the published value is the **median**
across runs. Token usage — input, output, cache-read, cache-write, and retrieved context — is
recorded per probe, the economic axis paired with accuracy.

## 6. Result format

[`data/latest.jsonl`](data/latest.jsonl) — one JSON object per `(probe × arm × model)`,
median-aggregated over the five runs:

```json
{
  "run_date": "2026-06-25", "product_version": "2026.06.25",
  "pack_id": "pack1", "probe_id": "p1-r1", "arm": "rlm",
  "model": "claude-haiku-4-5-20251001", "judge_model": "claude-haiku-4-5-20251001",
  "aa_score": 1.0, "rf_score": 1.0, "cr_score": 1.0, "sub_em_score": null,
  "input_tokens": 3570, "output_tokens": 198, "cache_read_tokens": 0,
  "cache_write_tokens": 0, "retrieved_tokens": 2741,
  "corpus_size": 100, "n_runs": 5, "probe_version": null
}
```

`corpus_size` is the number of distractor-padded decisions seeded for that row. Packs whose
corpus is padded to a fixed size carry that size (e.g. `100`); packs that run on their natural
seed set carry `corpus_size: null`. **Pack 1** is additionally run as a sweep across
`corpus_size ∈ {10, 100, 500, 1500}`, so tokens-processed-per-query can be measured against the
size of the seeded corpus — the consumption axis paired with accuracy.

## 7. Transparency boundary

**Published:** every probe, every seeded decision, the verbatim judge rubric, every score, and
the token counts ([`CORPUS.md`](CORPUS.md), [`SCORING.md`](SCORING.md),
[`data/latest.jsonl`](data/latest.jsonl)).

**Not published:** flmnt's retrieval algorithm, ranking weights, the embedding-and-traversal
engineering, and the retrieval system prompts. The runtime never leaves flmnt's infrastructure.

Where a seed system prompt's verbatim wording would describe retrieval-handling internals, the
public corpus shows a capability-level summary in its place and flags it as such (see the
naming note in [`CORPUS.md`](CORPUS.md)). This applies to the pack 8 prompt; the seeds and
probes for every pack, including pack 8, are published in full.
