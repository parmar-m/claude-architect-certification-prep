# CCA-P: Claude Certified Architect Professional

## The whole program at a glance

```mermaid
%%{init: {"themeVariables": {"git0": "#1A237E", "gitBranchLabel0": "#ffffff", "cScale0": "#303F9F", "cScale1": "#00838F", "cScale2": "#1565C0", "cScale3": "#4527A0", "cScale4": "#D32F2F", "cScale5": "#C2185B", "cScale6": "#6D4C41", "cScaleLabel0": "#ffffff", "cScaleLabel1": "#ffffff", "cScaleLabel2": "#ffffff", "cScaleLabel3": "#ffffff", "cScaleLabel4": "#ffffff", "cScaleLabel5": "#ffffff", "cScaleLabel6": "#ffffff"}}}%%
mindmap
  root(("🔵 CCA-P<br/>Professional"))
    D1 Solution Design and Architecture 17%
      Business problem → architecture
      Workflow vs agentic vs augmented LLM
      Multi-agent orchestration
    D2 Models, Prompting and Context Engineering 13%
      Model selection trade-offs
      Prompt caching, modular prompts, Skills
    D3 Integration 19%
      RAG pipelines, chunking, retrieval
      MCP vs API/CLI vs agent-to-agent
      AuthN/AuthZ, observability at scale
    D4 Evaluation, Testing and Optimization 16%
      Metrics: accuracy, latency, cost, safety
      Eval datasets, A/B testing
      Diagnosing failures
    D5 Governance, Safety and Risk 14%
      Guardrails and safety controls
      GDPR, HIPAA, FedRAMP
      Ethical AI
    D6 Stakeholder Communication and Lifecycle 14%
      Discovery and requirements
      Trade-off communication, SLAs
      Handoff and iteration
    D7 Developer Productivity and Enablement 7%
      Team tooling config
      AI-assisted workflows
```

## Domain chapters

| Chapter | Weight | Covers |
|---|---|---|
| [D1 Solution Design & Architecture](domains/d1-solution-design.md) | 17% | Business problem → architecture, workflow vs agentic vs augmented LLM, multi-agent design, value pillars |
| [D2 Models, Prompting & Context Engineering](domains/d2-models-prompting-context.md) | 13% | Model tier trade-offs, prompt architecture, prompt caching, token optimization, reuse (caching/modular/Skills) |
| [D3 Integration](domains/d3-integration.md) | 19% | RAG pipelines & retrieval strategy, MCP vs API/CLI vs agent-to-agent, authN/authZ & least privilege, observability |
| [D4 Evaluation, Testing & Optimization](domains/d4-evaluation-testing-optimization.md) | 16% | Metrics, eval datasets, LLM-as-judge, A/B testing, failure diagnosis, cost/latency levers |
| [D5 Governance, Safety & Risk](domains/d5-governance-safety-risk.md) | 14% | Layered guardrails, failure modes, HITL, GDPR/HIPAA/FedRAMP, ethical AI |
| [D6 Stakeholder Communication & Lifecycle](domains/d6-stakeholder-lifecycle.md) | 14% | Structured discovery, trade-off communication, SLAs, documentation, lifecycle phases |
| [D7 Developer Productivity & Enablement](domains/d7-dev-productivity-enablement.md) | 7% | Team tooling config, AI-assisted workflows, operational support |

Scenarios: **none**. The official guide defines no CCA-P scenario bank ([details](scenarios/README.md)). D1–D4 (65%) decide pass/fail; D5–D7 (35%) are the cheap points for CCA-F-trained candidates.
