# Changelog

All benchmark runs are listed here in reverse chronological order. No run is ever
deleted. Pre-fix failure runs are documented alongside their fix runs.

Results are in `data/YYYY-MM-DD.jsonl`. Each entry links to the relevant artifact.

---

## 2026-06-25 — economics + competitor baseline (GATE: PASS)

**Product version:** 2026.06.25
**Models:** claude-opus-4-8, claude-haiku-4-5-20251001
**Judge model:** claude-haiku-4-5-20251001
**Runs per probe:** 5 (median reported)
**Data:** `data/latest.jsonl` — the prior run is preserved verbatim in
`data/2026-06-22-0.jsonl` (no run is ever deleted).

This run adds a `flat` arm (vector-similarity retrieval, no graph traversal) and an
economic axis (tokens processed per query as the seeded corpus scales), widens the suite
to 8 packs and 5 arms, and reports two model sizes.

### Results — median AA, both models unless noted

| Pack | Scenario | cold | warm | flat | rlm | oracle |
|---|---|---|---|---|---|---|
| 1 | Core context recovery | 0.00 | 0.00 | 1.00 | 1.00 | 1.00 |
| 2 | Decision supersession (recency) | 0.00 | 0.00 | 1.00 | 1.00 | 1.00 |
| 3 | Causal traversal | 0.00 | 0.00 | 1.00 | 1.00 | 1.00 |
| 4 | Multi-session handoff | 0.00 | 1.00 | 1.00 | 1.00 | 1.00 |
| 5 | Project isolation | 0.00 | 1.00 | 1.00 | 1.00 | 1.00 |
| 6 | Abstention & hallucination guard | 0.00 | 1.00 | 1.00 | 1.00 | 1.00 |
| 7 | Multi-hop causal chain | 0.00 | 0.00 | 0.95–1.00 | 0.95 | 0.95 |
| 8 | Structural supersession | 0.00 | 0.00 | **0.00** | **1.00** | 1.00 |

Opus and Haiku produce the same median scores on every pack except pack 7, where the flat
arm reads 1.00 on Opus and 0.95 on Haiku.

### Observations

- On packs 1–7, `rlm` and `flat` are equal (`rlm − flat = 0.00`); on pack 7 the flat arm
  is 0.05 higher than rlm on Opus.
- On pack 8, `flat` scores 0.00 and `rlm` scores 1.00 on both models. The stale decision
  in pack 8 carries no textual staleness marker; the flat arm returns it and the rlm arm
  returns the superseding decision.
- `warm` scores 1.00 on packs 4–6 (natural corpus) and 0.00 on packs 1, 2, 3, 7 (corpus
  padded to 100 distractors). `rlm` scores 1.00 on all eight packs. The v0.1 baseline's
  ~0.99 warm scores were recorded at smaller corpus sizes.

### Consumption — pack 1 corpus-size sweep

Tokens processed per query (input + cache-read), `claude-haiku-4-5`, comparing `rlm`
(bounded retrieval) against injecting the full corpus:

| Seeded corpus | rlm | full-corpus inject | difference |
|---|---|---|---|
| 10 | 4,826 | 1,511 | −219% |
| 100 | 6,816 | 4,204 | −62% |
| 500 | 10,734 | 15,693 | +32% |
| 1,500 | 8,054 | 44,670 | +82% |

The two cross between 100 and 500 seeded decisions.

### Fairness fixes applied before this run

Found in a pre-run audit and documented by class and impact.

1. **Competitor arm ran an unequal prompt.** The `flat` arm's task framing lacked the
   premise-rejection and abstention guidance the other arms received. Fixed: every arm now
   receives identical task framing, differing only in which retrieval tool is offered.

2. **Distractor padding produced near-duplicates.** The synthetic decisions used to scale
   a corpus could collide into near-identical entries, which crowded the competitor's
   vector retrieval unfairly. Fixed to generate distinct distractors across topic, choice,
   and rationale.

3. **A retrieval serialization defect dropped a recorded signal** before it reached the
   model on the supersession path. Fixed. (Bug class only — the affected signal and its
   handling are product internals and are not described here.)

4. **Cost accounting is now cache-aware.** Cache-read and cache-write tokens are priced
   separately from fresh input, so the economic table reflects real metered cost.

### Differences from the v0.1 baseline

Pack 1–7 warm/rlm numbers differ from the May baseline for two reasons: (a) the fairness
fixes above, and (b) corpus scaling, which exposes snapshot-tier truncation at larger
corpus sizes. The prior run is preserved in `data/2026-06-22-0.jsonl`.

---

## 2026-05-18 — v0.1 baseline (GATE: PASS)

**Product version:** 0.1.0
**Models:** claude-opus-4-7, claude-haiku-4-5-20251001
**Judge model:** claude-haiku-4-5-20251001
**Composite warm-arm AA:** 0.993 (both models)

### Results

| Pack | Scenario | Opus-4-7 | Haiku-4-5 | Gate |
|---|---|---|---|---|
| 1 | Simple recall | 1.00 | 1.00 | PASS |
| 2 | Recency & versioning | 1.00 | 1.00 | PASS |
| 3 | Cross-stream synthesis | 0.98 | 1.00 | PASS |
| 4 | Long-stream retrieval | 0.99 | 1.00 | PASS |
| 5 | Cross-session isolation | 1.00 | 1.00 | PASS |
| 6 | Keyframe recall | 1.00 | 1.00 | PASS |
| 7 | Multi-hop causal (RLM) | 0.97 | 0.97 | PASS |
| **Composite** | | **0.993** | **0.993** | **PASS** |

### Known failures resolved in this run

#### Pack 5 — Probe phrasing caused literal stream_id passthrough

**Pre-fix behavior:** A probe phrased around "the Beta project" caused the model to pass
`stream_id="Beta"` as a literal string to context retrieval, breaking stream resolution.
The probe was designed to test cross-session isolation using project name as the
discriminator; the phrasing created an ambiguity between the project name as natural
language and as a resolvable stream identifier.

**Resolution:** Probe rephrased to remove the ambiguity. Pre-fix results for Pack 5 are
preserved in the result artifact under `arm="warm_preflight"`.

**Date resolved:** 2026-05-18

---

#### Pack 7 — Three Engine bugs caused warm/cold/rlm arm AA=0.00

**Pre-fix behavior:** Warm, cold, and RLM arms all scored AA=0.00. Oracle arm was
unaffected (AA=0.97), confirming the answers existed in the stream and the judge model
was functioning correctly.

Three bugs were identified in the Engine layer:

1. **Keyframe 404 crash** — The Engine requested a keyframe that did not exist for the
   given session configuration, producing a 404 error that aborted retrieval entirely.

2. **Markdown fence stripping in JSON.parse** — Responses containing markdown code fences
   were passed directly to `JSON.parse` without stripping the fence markers, causing a
   parse failure and returning an empty context to the model.

3. **Raw array response shape misread** — A response returning a raw JSON array was
   expected to be an object with a data key; the misread produced an undefined context
   instead of the array contents.

All three bugs were in the Engine layer, not in the retrieval algorithm. The oracle arm's
unaffected score (0.97) is the proof: the data was in the stream and the judge could
score it correctly — the retrieval pipeline was failing before the algorithm ran.

**Resolution:** All three Engine bugs fixed. Post-fix rerun: warm=0.97, rlm=0.97,
oracle=0.97. Gate PASS.

**Date resolved:** 2026-05-18
