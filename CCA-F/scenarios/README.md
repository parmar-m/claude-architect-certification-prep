# CCA-F Scenario Bank ➜ Domains Tested

Each exam uses **4 of these 6 scenarios**. A scenario can test two or three domains.

```mermaid
flowchart LR
    S1["① Customer Support Resolution Agent"] --> D1["D1 Agentic Architecture"] & D2["D2 Tool Design & MCP"] & D5["D5 Context & Reliability"]
    S2["② Code Generation with Claude Code"] --> D3["D3 Claude Code"] & D5
    S3["③ Multi-Agent Research System"] --> D1 & D2 & D5
    S4["④ Developer Productivity"] --> D2 & D3 & D1
    S5["⑤ Claude Code for CI/CD"] --> D3 & D4["D4 Prompting & Output"]
    S6["⑥ Structured Data Extraction"] --> D4 & D5
    classDef scen fill:#455A64,color:#fff,stroke:#263238,stroke-width:2px
    classDef d1 fill:#512DA8,color:#fff,stroke:#311B92,stroke-width:2px
    classDef d2 fill:#00796B,color:#fff,stroke:#004D40,stroke-width:2px
    classDef d3 fill:#1976D2,color:#fff,stroke:#0D47A1,stroke-width:2px
    classDef d4 fill:#E64A19,color:#fff,stroke:#BF360C,stroke-width:2px
    classDef d5 fill:#388E3C,color:#fff,stroke:#1B5E20,stroke-width:2px
    class S1,S2,S3,S4,S5,S6 scen
    class D1 d1
    class D2 d2
    class D3 d3
    class D4 d4
    class D5 d5
```

D5 appears in four scenarios despite its 15% weight, so study it across several contexts.

## Scenario deep-dives

Each file contains the official brief, a reference architecture, tested patterns, and practice questions.

| Scenario | Official sample questions |
|---|---|
| [① Customer Support Resolution Agent](s1-customer-support-agent.md) | Q1–Q3 |
| [② Code Generation with Claude Code](s2-code-generation.md) | Q4–Q6 |
| [③ Multi-Agent Research System](s3-multi-agent-research.md) | Q7–Q9 |
| [④ Developer Productivity with Claude](s4-developer-productivity.md) | none (practice only) |
| [⑤ Claude Code for CI/CD](s5-ci-cd.md) | Q10–Q12 |
| [⑥ Structured Data Extraction](s6-structured-data-extraction.md) | none (practice only) |
