# P-D6 · Stakeholder Communication & Lifecycle Management (14%)

Gathering requirements, explaining trade-offs, setting expectations, documenting decisions, and managing a solution after launch.

**Source:** official [CCA-P Exam Guide](../../official-exam-guides/cca-p-exam-guide.pdf), Domain 6 objectives; prep course "Stakeholder Engagement, Lifecycle & GTM".

---

## The solution lifecycle

```mermaid
flowchart LR
    DISC["1️⃣ Discovery: structured requirements: users · data · constraints · success criteria · risk appetite"] --> DES["2️⃣ Design: architecture + trade-offs communicated & signed off"]
    DES --> BUILD["3️⃣ Build & validate: prototype → evals → stakeholder feedback loops"]
    BUILD --> HAND["4️⃣ Handoff: docs · runbooks · implementation guidance · enablement"]
    HAND --> MON["5️⃣ Monitoring: SLAs · quality dashboards · incident paths"]
    MON --> IT["6️⃣ Iteration: feedback → backlog → prompt/model/data updates"]
    IT --> DISC
    classDef ph fill:#C2185B,color:#fff,stroke:#880E4F,stroke-width:2px
    class DISC,DES,BUILD,HAND,MON,IT ph
```

## Structured discovery

A requirement must be testable before it can guide a design. Record the business goal, constraints, assumptions, and evidence needed to demonstrate success.

**Key points:**

- Gather requirements in order: business goal → users and volume → data and sensitivity → integrations → latency and cost limits → compliance → success measures.
- Agree on success measures before building. The accuracy target set during discovery becomes the evaluation target in [P-D4](d4-evaluation-testing-optimization.md).
- Ask about hidden constraints early. See [F-D3 Sec. 3.5](../../CCA-F/domains/d3-claude-code.md).
- Record four categories: **outcomes**, **prohibited actions and approval boundaries**, **cost/latency/volume limits**, and **required evidence**. For each requirement, capture the constraint, design implication, and any assumption.

**Feasibility:** Assess whether next-token prediction suits the task, required knowledge is available, working memory is sufficient, and instructions can steer behavior reliably. Check these limits against cost and latency constraints; document every condition the verdict depends on.

| Verdict | Meaning | Must document |
|---|---|---|
| Feasible as scoped | Task fits model capabilities; cost and p95 latency meet limits | Assumptions and when to revalidate them |
| Feasible with constraints | Works only under specific enforced conditions | Each condition and the failure if it is violated |
| Not feasible | A property limitation can't be compensated within scope/budget | Which constraint disqualifies it, and any scope change that would flip the verdict |

## Communicating trade-offs

```mermaid
flowchart TD
    TO["Architectural trade-off"] --> AUD{Audience}
    AUD -- "executives" --> EX["Business terms: value pillar impact, risk, cost curve, time-to-market"]
    AUD -- "engineering" --> EN["Technical terms: latency budgets, failure modes, integration surface"]
    AUD -- "legal / compliance" --> LG["Data flows, regimes, controls, audit evidence"]
    EX & EN & LG --> REC["Recommendation + alternatives + reasons for the decision"]
    classDef aud fill:#C2185B,color:#fff,stroke:#880E4F,stroke-width:2px
    classDef n fill:#455A64,color:#fff,stroke:#263238,stroke-width:2px
    class EX,EN,LG,REC aud
    class TO,AUD n
```

**Key points:**

- Recommend one option and explain why. Record the context, alternatives, decision, and effects in an architecture decision record (ADR).
- State the **benefit, limitation, and cost of reversing the decision**; include its compliance impact.
- Use measured accuracy ranges and explain error handling. Do not promise perfect results. Allow time for evaluation and improvement.
- **Partner Track (not tested by the Architect exam):** Design demos around the buyer's real scenario, identify limitations first, and bring requirements, candidate patterns, and open questions to joint scoping.

## SLAs & feedback loops

- An AI SLA should cover **latency** (p50/p95), **availability**, a measured **quality floor**, a **cost ceiling**, and incident response times.
- Review quality dashboards and sample outputs with stakeholders. Add user feedback to the evaluation set. Publish changes to prompts and models because they can change system behavior.
- Map each signal to a trigger, action, and owner. Set SLA thresholds from user expectations, business needs, and evals; schedule regulated checkpoints independently of alerts.

## Documentation & handoff

**Handoff package:** Architecture and data-flow diagrams · versioned prompt and configuration list · runbooks with common failures and escalation paths · evaluation baselines and rerun instructions · known limits. Include training and a defined support period. See [P-D7](d7-dev-productivity-enablement.md).

Include ADRs and the control register with owners and evidence. **Completeness test:** Can an architect who was not in the room make a safe change using the document?

For outcomes, record scope, before-and-after business metrics, the auditable control, measurement owner, and reuse potential to support an expansion decision.

---

## Extra practice (unofficial)

*No official sample questions exist for this domain; the CCA-P Exam Guide's 3 published samples cover Domains 2–4. Practice questions below are unofficial, written against this domain's official objectives.*

**P1.** You need to explain why you chose a smaller, cheaper model tier for a high-volume classification step to three audiences: the CFO, the engineering team, and legal/compliance. What's the best approach?

- **A.** Send the same detailed technical write-up to all three audiences to ensure consistency.
- **B.** Tailor the same decision to each audience's concerns (cost/value framing for the CFO, latency/failure-mode detail for engineering, data-flow/compliance framing for legal), each with one clear recommendation.
- **C.** Only inform engineering, since model selection is a technical decision.
- **D.** Present the CFO and legal team with multiple options and no recommendation, to avoid overstepping into their domain.

<details><summary>Answer & rationale</summary>

**B.** Each audience needs different details, but all should receive the same recommendation and rationale.
</details>

**P2.** A stakeholder asks for "a chatbot that answers customer questions using our knowledge base," with no further detail. Before proposing an architecture, what should you do?

- **A.** Immediately propose a RAG architecture, since that's the standard pattern for knowledge-base Q&A.
- **B.** Run structured discovery: clarify users/volumes, data sources and sensitivity, integration points, latency/cost budgets, compliance constraints, and measurable success criteria.
- **C.** Build a quick prototype first, then ask for requirements based on stakeholder reactions to it.
- **D.** Ask only about the technology stack the knowledge base is stored in.

<details><summary>Answer & rationale</summary>

**B.** Data sensitivity, compliance, scale, and success measures can change the design. Confirm them before selecting an architecture.
</details>

**P3.** A production system's SLA currently states only "the system will respond within 2 seconds." Six months in, stakeholders are blindsided by a week where responses stayed fast but answer quality clearly degraded after a prompt change, with no agreed process to report or escalate it. What's missing from the SLA?

- **A.** Nothing; a 2-second latency target is a complete SLA for an AI system.
- **B.** A quality floor (eval-measured accuracy) and incident escalation/response times, alongside the existing latency target.
- **C.** A stricter latency target, e.g. 1 second instead of 2.
- **D.** A clause stating the system will always be 100% accurate.

<details><summary>Answer & rationale</summary>

**B.** The SLA covers latency but not quality or escalation. A quality floor and response times would make the regression visible and actionable.
</details>

**P4.** A contracting team finishes building a Claude-based system and hands it off to the client's internal team with a single email summarizing what was built. Three weeks later, the client's team can't explain why a specific prompt behaves the way it does, doesn't know how to re-run the evals, and has no idea who to contact for a production incident. What was missing from the handoff?

- **A.** A more detailed email.
- **B.** Architecture diagrams and data-flow maps, a versioned prompt/config inventory, runbooks with escalation paths, eval baselines and how to re-run them, and stated known limitations.
- **C.** A longer support contract.
- **D.** A recorded video walkthrough of the demo.

<details><summary>Answer & rationale</summary>

**B.** A useful handoff gives the new team durable references: system diagrams, versioned configuration, evaluation instructions, runbooks, and support contacts.
</details>

**P5.** Six months after launch, a team notices gradual quality decay as customer queries have shifted toward new product lines the original eval set didn't cover. Stakeholder reviews of quality dashboards are already happening quarterly. What lifecycle phase does this call for next?

- **A.** Discovery: start over with new requirements gathering from scratch.
- **B.** Iteration: feed the observed drift back into updated eval cases and prompt/data updates, looping back toward design only as far as the drift requires.
- **C.** Handoff: re-hand off the system to a different team.
- **D.** Build & validate: re-prototype the system from the ground up.

<details><summary>Answer & rationale</summary>

**B.** Monitoring found drift, so the team should update the evaluation set and then adjust prompts, models, or data as needed.
</details>

## Exam focus

| Cue | Direction |
|---|---|
| "CEO asks why not the cheaper option" | Value-pillar framing, risk trade-off, recommendation with rationale |
| "Stakeholders expect 100% accuracy" | Reset expectations: measured ranges + error handling + HITL |
| "What goes in the SLA?" | Latency + availability + quality floor + incident response |
| "Team inherits the system" | Runbooks, prompt inventory, eval baselines, limitations |
| "Requirements keep shifting" | Structured discovery artifacts + agreed success criteria |

**Practice:** prep course module ④ (178 min) · write an ADR for one of your own designs.
