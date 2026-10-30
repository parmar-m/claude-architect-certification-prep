# P-D4 · Evaluation, Testing & Optimization (16%)

Defining metrics, building evaluation sets, testing changes, finding failure causes, and improving token use, latency, and cost.

**Builds on CCA-F:** [F-D4 Sec. 4.4–4.6](../../CCA-F/domains/d4-prompting-structured-output.md) (validation loops, review architectures) · [F-D5 Sec. 5.5](../../CCA-F/domains/d5-context-reliability.md) (confidence calibration).
**Source:** official [CCA-P Exam Guide](../../official-exam-guides/cca-p-exam-guide.pdf), Domain 4 objectives.

---

## The evaluation loop

Evaluation measures behavior against a defined target. Monitoring checks whether that behavior holds under production inputs; controlled experiments test whether a change improves it.

```mermaid
flowchart LR
    DEF["1️⃣ Define metrics: accuracy · latency · cost · safety · security"] --> DS["2️⃣ Build eval dataset: golden sets · edge cases · real production samples"]
    DS --> RUN["3️⃣ Run evals: code graders · LLM-as-judge · human review (mixed methods)"]
    RUN --> AB["4️⃣ A/B test changes: prompt / model / retrieval variants"]
    AB --> MON["5️⃣ Monitor production: logging · dashboards · sampled review · drift alerts"]
    MON --> DEF
    classDef step fill:#4527A0,color:#fff,stroke:#311B92,stroke-width:2px
    class DEF,DS,RUN,AB,MON step
```

**Key points:**

- **Define metrics before optimizing.** Choose metrics for the stakeholder's concern: safety for compliance, latency for users, or cost for finance.
- **Production readiness:** A demo does not establish performance at scale. Evaluate quality, estimate cost and latency at expected volume, and document failure modes with mitigations such as retries, fallbacks, and circuit breakers.
- Write evals before production code, keep golden sets current, and gate model swaps and prompt revisions on results.
- **Use several evaluation methods:** fixed graders for structure and facts, LLM judges with calibrated rubrics for open-ended quality, and human review for high-risk samples.
- Evaluation sets need **edge cases and real production samples**. Refresh them as usage changes. Overall scores can hide failures, so also measure each input type ([F-D5 Sec. 5.5](../../CCA-F/domains/d5-context-reliability.md)).

### A/B testing

Change **one variable at a time**, hold the eval set constant, and check previously passing segments for regressions. State a falsifiable hypothesis, such as increasing task success by 5% without degrading p95 latency.

| Component | Requirement | Risk if missing |
|---|---|---|
| Hypothesis | Falsifiable; names treatment, metric, and threshold | No defined condition for success or failure |
| Treatment/control assignment | Random and consistent per user or session | Groups may not be comparable |
| Primary metric | Choose before the experiment | Selecting a favorable metric afterward biases the conclusion |
| Sample size | Size for the minimum detectable effect | An underpowered test cannot distinguish signal from noise |

Use **live A/B** for real user signals when traffic is sufficient and exposure to a worse version is acceptable. Use **shadow testing** for high-stakes, regulated, or low-traffic settings: run in parallel, log only, and score offline.

## Failure diagnosis

Start with the changed component and trace its inputs and outputs. Similar symptoms can originate in retrieval, prompts, model behavior, or orchestration.

```mermaid
flowchart TD
    SYM["🚨 Quality regression"] --> Q1{What changed?}
    Q1 -- "documents / index refreshed" --> RET["🔎 Retrieval first: stale/irrelevant chunks, broken re-index, mismatched embeddings"]
    Q1 -- "prompt edited" --> PR["📝 Prompt failure: criteria drift, lost instructions, format break"]
    Q1 -- "model / tier swapped" --> MM["🤖 Model mismatch: task too hard for tier, or behavior shift on upgrade"]
    Q1 -- "nothing (gradual)" --> DRIFT["📈 Input drift: new doc types / query patterns outside eval coverage"]
    SYM --> HAL{"Confident but wrong (hallucination)?"}
    HAL -- "grounded app" --> RET
    HAL -- "schema fields invented" --> SCH["Nullable fields + validation (F-D4 Sec. 4.3)"]
    classDef diag fill:#4527A0,color:#fff,stroke:#311B92,stroke-width:2px
    classDef n fill:#455A64,color:#fff,stroke:#263238,stroke-width:2px
    class RET,PR,MM,DRIFT,SCH diag
    class SYM,Q1,HAL n
```

| Failure | Response |
|---|---|
| Prompt failure | Identify missing or conflicting instructions; revise them and rerun affected evals |
| Hallucination | Ground, retrieve, or verify rather than rely on stronger instructions |
| Model mismatch | Select a model using evals as a gate |
| Orchestrator-workers failure | Trace across the orchestrator and workers |

## Optimization levers

Optimize the measured bottleneck while preserving the quality target. Cost, wall-clock latency, and perceived latency are separate metrics.

| Lever | Benefit | Constraint |
|---|---|---|
| Prompt caching (stable prefix) | Cost + time-to-first-token | Requires stable ordering ([P-D2](d2-models-prompting-context.md)) |
| Model tiering / routing | Cost at scale | Validate per-tier quality with evals |
| Batch API for offline volume | 50% cost | No latency SLA, never for blocking flows ([F-D4 Sec. 4.5](../../CCA-F/domains/d4-prompting-structured-output.md)) |
| Trim context / retrieve relevant material | Cost + attention to relevant facts | Preserve provenance and citations |
| Streaming | Perceived latency | Doesn't reduce cost |
| Parallel tool/retrieval calls | Wall-clock latency | Rate limits, ordering dependencies |

## Monitoring & observability

- Cover request tracing, metric aggregation, anomaly detection, and change attribution. Retain per-request details alongside dashboards: aggregates can hide a costly minority of calls.
- Log the prompt version, model, tokens, cost, stage latency, retrieval results and scores, tool calls, output, and validation result for each request.
- **Use a shared trace ID across orchestrator, worker, and tool hops**; alert on drift in cost, latency, refusal rate, validation-failure rate.
- Add failures found in sampled production review to the eval set.
- **Discernment:** Classify output as acceptable, needing revision, or needing override; feed decisions into evals and monitoring. Metrics alone do not establish output quality.
- At design time, map technical metrics to stakeholder business metrics, such as task success rate to first-contact resolution.

---

## Official sample question

*From the [CCA-P Exam Guide](../../official-exam-guides/cca-p-exam-guide.pdf), Sec. 8, Sample 3.*

A RAG system suddenly returns confident but incorrect answers after a document refresh, while latency and model version are unchanged. What is the most likely first place to investigate?

- **A.** The model weights have silently changed.
- **B.** The retrieval/indexing step is returning irrelevant or stale chunks.
- **C.** The temperature setting is too low.
- **D.** The context window has shrunk.

<details><summary>Answer & rationale</summary>

**B.** The document refresh is the only change, so retrieval may be returning poor context because of a broken index or mismatched embeddings. The other options are not linked to the refresh.
</details>

## Extra practice (unofficial)

**P1.** After a routine model upgrade (same prompts, same retrieval pipeline, same infra), an eval suite shows accuracy unchanged, but a new failure pattern appears: the model now occasionally refuses benign requests it used to handle. What's the most likely first place to investigate?

- **A.** The retrieval/indexing pipeline.
- **B.** The new model version's default safety/refusal behavior interacting with the existing system prompt's instructions.
- **C.** The temperature setting.
- **D.** The eval dataset's ground-truth labels, which may be stale.

<details><summary>Answer & rationale</summary>

**B.** Only the model changed. New refusals therefore point to its safety behaviour interacting with the existing prompt. Retrieval did not change, temperature does not explain refusals, and stale labels would affect measured accuracy.
</details>

**P2.** A team's only eval for a customer-facing drafting agent is automated exact-match string comparison against reference answers. Drafts that are correct but phrased differently are marked as failures, and the team has no way to catch subtly unhelpful (but string-different) responses. What should they add?

- **A.** Increase the size of the exact-match reference set.
- **B.** Add LLM-as-judge scoring against a rubric, plus periodic human review, alongside the existing automated check.
- **C.** Lower the passing threshold on the exact-match eval.
- **D.** Remove the automated eval and rely on human review alone.

<details><summary>Answer & rationale</summary>

**B.** Exact matching cannot recognise correct answers written differently. Add an LLM judge and human review while keeping the automated check.
</details>

**P3.** A team wants to cut their agent's per-request cost. Logs show 70% of requests are simple FAQ-style questions, currently served by the same large model tier used for complex multi-step troubleshooting. What's the most directly justified optimization?

- **A.** Reduce `max_tokens` across all requests.
- **B.** Route the FAQ-style requests to a smaller/cheaper tier, informed by measured task difficulty, and reserve the large tier for the complex cases.
- **C.** Enable prompt caching on the large tier for all requests.
- **D.** Batch all requests overnight using the Batches API.

<details><summary>Answer & rationale</summary>

**B.** Most requests are simple and do not need the expensive model. A may cut off complex answers. C does not fix poor model routing. D would remove the real-time response that live FAQ users need.
</details>

**P4.** A team simultaneously changes the system prompt, switches to a new model version, and adjusts retrieval top-k, then reruns their eval suite and sees a 6% accuracy improvement. What's the problem with this result?

- **A.** There is no problem; the eval suite confirmed an improvement.
- **B.** Three variables changed at once, so it's impossible to attribute the 6% gain to any specific change, or to see whether one change is quietly hurting performance while another masks it.
- **C.** The eval suite should have been made harder before rerunning.
- **D.** The model version change alone is definitely responsible, since model upgrades usually help the most.

<details><summary>Answer & rationale</summary>

**B.** Change one variable at a time and keep the evaluation set fixed. With three changes, you cannot identify the cause of the gain or see whether one change caused a hidden regression.
</details>

**P5.** A team needs to process 50,000 documents overnight with no one waiting on individual results, and separately runs a live chat feature where users watch responses arrive. Both currently use the same real-time API call pattern. What should change?

- **A.** Add streaming to both, to improve perceived responsiveness.
- **B.** Move the overnight job to the Batches API for cost savings (no latency SLA there), and keep or add streaming only on the live chat feature for perceived latency.
- **C.** Move both to the Batches API, since it's cheaper.
- **D.** Add prompt caching to both, since caching always reduces cost regardless of workload shape.

<details><summary>Answer & rationale</summary>

**B.** The Batch API fits offline work where no one waits for a result. Streaming helps the live chat feel faster. A adds no value to the overnight job, C breaks live response, and D does not address the wrong API choice.
</details>

## Exam focus

| Cue | Direction |
|---|---|
| "How do we know it's good enough to ship?" | Eval dataset + defined metrics first |
| "Confident wrong answers after refresh" | Retrieval/indexing |
| "Judge open-ended quality at scale" | LLM-as-judge + calibration + human spot-checks |
| "Cost exploding at volume" | Caching → tiering → batch (in that order of ease) |
| "Which variant is better?" | A/B on a fixed eval set, one variable |

**Practice:** [claude-cookbooks `evals` + `tool_evaluation` + `observability`](https://github.com/anthropics/claude-cookbooks) · [anthropics/courses prompt evaluations](https://github.com/anthropics/courses).
