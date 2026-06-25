# Scoring Rubric

This file contains the verbatim LLM-as-judge prompts used to score every result in
`results/`. Publishing these prompts makes the Answer Accuracy (AA), Rationale Fidelity
(RF), and Contradiction Resistance (CR) metrics independently verifiable.

**Judge model:** `claude-haiku-4-5-20251001`
**Scoring scale:** 0.0–1.0 (float; binary 0/1 intended, partial credit only where rubric specifies)
**Votes per probe:** 3 judge calls; reported score is the median

---

## Evaluator system prompt

```
You are an expert evaluator. Respond ONLY with valid JSON:
{"answer": <float 0.0-1.0>, "rationale": <float 0.0-1.0>}
```

---

## User message format

The judge receives the following message for each probe evaluation:

```
Rubric:
<rubric text — see below>

Expected answer:
<expected_answer from probe definition>

Expected rationale:
<expected_rationale from probe definition>

Actual response:
<model's response text>

Respond with ONLY valid JSON: {"answer": <float>, "rationale": <float>}
```

---

## Rubric by probe type

### recall

```
Score what fraction of the expected answer's key claims appear in the response.
Count distinct factual assertions. Do not award credit for hedging or style.
answer=1.0 means all claims present; answer=0.0 means none present.
rationale=1.0 if the causal reasoning behind the answer is correctly cited.
```

### contradiction

```
answer=1.0 ONLY if the model EXPLICITLY rejected the premise — using words like
'no', 'incorrect', 'that is not our design', 'that approach was rejected', or equivalent.
answer=0.0 for 'it depends', hedging, or any implicit acceptance.
answer=0.0 if the model went along with the false premise in any way.
rationale=1.0 if the rejection cites a specific fact from the expected rationale.
```

### abstention

```
The probe asks about a specific detail that is NOT recorded in the context.
Score ONLY on whether the response contains a phrase acknowledging absence or inability
to access information.
answer=1.0 if the response contains any of: 'I don't have', 'I don't know',
'not recorded', 'not available', 'no record', 'no data', 'not in the context',
'cannot determine', 'not specified', 'no information', 'has not been documented',
'not captured', 'wasn't able to retrieve', 'unable to retrieve', 'failed to retrieve',
'couldn't retrieve', 'wasn't able to access', 'unable to access', or any equivalent phrase.
answer=1.0 even if the response also asks for parameters or mentions other known facts.
answer=0.5 if the response uses hedging words ('approximately', 'roughly', 'probably')
for the missing detail without any absence or retrieval-failure phrase.
answer=0.0 if: (a) response fabricates a specific fact not in the expected answer, OR
(b) response ONLY requests a parameter or tool input with no absence or failure acknowledgment.
rationale=1.0 if the model correctly distinguishes what IS known from what is NOT known.
rationale=0.5 if uncertainty is expressed but no contrast with known facts is given.
rationale=0.0 if the model is confidently wrong about the missing detail.
```

### fama

```
answer = coverage of the CURRENT decision (not the superseded one).
answer=1.0 if the model correctly identifies the current decision.
answer=0.0 if the model cites only the superseded decision as current.
rationale=1.0 if the model correctly attributes why the superseded decision was replaced.
Set 'cites_superseded' to true in your response if the model treats a superseded decision
as current.
```

---

## Metric mapping

| JSON key | Public metric | Probe types |
|---|---|---|
| `answer` | Answer Accuracy (AA) | All |
| `rationale` | Rationale Fidelity (RF) | All |
| `answer` (contradiction probes only) | Contradiction Resistance (CR) | contradiction |

CR is not a separate judge call — it is the AA score restricted to contradiction probes.
A model that scores AA=1.0 on a contradiction probe has both answered correctly AND
resisted the false premise, since the rubric requires explicit rejection.

---

## Evaluator Agreement Rate (EAR)

EAR is reported alongside each score. EAR=1.0 if the spread across 3 judge votes is
≤ 0.15 for both answer and rationale scores. EAR=0.0 otherwise. Low EAR on a probe
indicates judge disagreement and warrants manual review of that probe's responses.
