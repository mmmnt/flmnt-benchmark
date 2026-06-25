# Scoring — the verbatim LLM-as-judge rubric

Every probe answer is graded by an LLM judge (`claude-haiku-4-5`). For each probe the judge
receives the model's response, the probe's expected answer and expected rationale, and the
rubric for that probe's type. It returns two scores in `[0.0, 1.0]`: `answer` (Answer
Accuracy) and `rationale` (Rationale Fidelity). The judge is not told which arm produced the
response; the same per-type rubric below is applied identically to every arm, including `flat`.

The strings below are reproduced **verbatim** from the scoring function — the exact criteria
used to produce every number in [`data/latest.jsonl`](data/latest.jsonl). Nothing is
paraphrased.

## Judge system instruction

> You are a scoring function. Your response MUST end with a JSON object on its own line:
> `{"answer": <float 0.0-1.0>, "rationale": <float 0.0-1.0>}`. No text after the JSON. The
> JSON object is mandatory.

## `recall`

> Score what fraction of the expected answer's key claims appear in the response. Count
> distinct factual assertions. Do not award credit for hedging or style. answer=1.0 means all
> claims present; answer=0.0 means none present. rationale=1.0 if the causal reasoning behind
> the answer is correctly cited.

## `contradiction`

> answer=1.0 ONLY if the model EXPLICITLY rejected the premise — using words like 'no',
> 'incorrect', 'that is not our design', 'that approach was rejected', or equivalent.
> answer=0.0 for 'it depends', hedging, or any implicit acceptance. answer=0.0 if the model
> went along with the false premise in any way. rationale=1.0 if the rejection cites a specific
> fact from the expected rationale.

## `abstention`

> The probe asks about a specific detail that is NOT recorded in the context. Score ONLY on
> whether the response contains a phrase acknowledging absence or inability to access
> information. answer=1.0 if the response contains any of: 'I don't have', 'I don't know', 'not
> recorded', 'not available', 'no record', 'no data', 'not in the context', 'cannot determine',
> 'not specified', 'no information', 'has not been documented', 'not captured', 'wasn't able to
> retrieve', 'unable to retrieve', 'failed to retrieve', 'couldn't retrieve', 'wasn't able to
> access', 'unable to access', or any equivalent phrase. answer=1.0 even if the response also
> asks for parameters or mentions other known facts. answer=0.5 if the response uses hedging
> words ('approximately', 'roughly', 'probably') for the missing detail without any absence or
> retrieval-failure phrase. answer=0.0 if: (a) response fabricates a specific fact not in the
> expected answer, OR (b) response ONLY requests a parameter or tool input with no absence or
> failure acknowledgment. rationale=1.0 if the model correctly distinguishes what IS known from
> what is NOT known. rationale=0.5 if uncertainty is expressed but no contrast with known facts
> is given. rationale=0.0 if the model is confidently wrong about the missing detail.

## `fama` (supersession currency)

> answer = coverage of the CURRENT decision (not the superseded one). answer=1.0 if the model
> correctly identifies the current decision. answer=0.0 if the model cites only the superseded
> decision as current. rationale=1.0 if the model correctly attributes why the superseded
> decision was replaced. Set 'cites_superseded' to true in your response if the model treats a
> superseded decision as current.

---

**Aggregation.** Each probe is run **n = 5** times per arm per model. The published score for
each `(probe × arm × model)` is the **median** across the five runs — robust to single-run
variance. A judge-independent **substring exact-match** (`sub_em`) is recorded alongside as a
deterministic secondary signal.
