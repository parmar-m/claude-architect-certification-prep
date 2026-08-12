# CCA-F: Claude Certified Architect Foundations

## The whole program at a glance

```mermaid
%%{init: {"themeVariables": {"git0": "#004D40", "gitBranchLabel0": "#ffffff", "cScale0": "#512DA8", "cScale1": "#00796B", "cScale2": "#1976D2", "cScale3": "#E64A19", "cScale4": "#388E3C", "cScaleLabel0": "#ffffff", "cScaleLabel1": "#ffffff", "cScaleLabel2": "#ffffff", "cScaleLabel3": "#ffffff", "cScaleLabel4": "#ffffff"}}}%%
mindmap
  root(("🟢 CCA-F: Foundations"))
    D1 Agentic Architecture and Orchestration 27%
      Agentic loops and stop_reason
      Coordinator-subagent patterns
      Task decomposition
      Hooks and enforcement
      Session state and forking
    D2 Tool Design and MCP Integration 18%
      Tool interface design
      MCP servers, resources, prompts
      Error handling
    D3 Claude Code Configuration and Workflows 20%
      CLAUDE.md hierarchy
      Slash commands and skills
      Plan mode
      CI/CD integration
    D4 Prompt Engineering and Structured Output 20%
      System prompts
      Few-shot examples
      JSON schemas and validation
    D5 Context Management and Reliability 15%
      Context windows
      Summarization
      Escalation and human-in-the-loop
```

## Domain chapters

| Chapter | Weight | Covers |
|---|---|---|
| [D1 Agentic Architecture & Orchestration](domains/d1-agentic-architecture.md) | 27% | Agentic loop & stop_reason, coordinator–subagent, Task tool & context passing, hooks, decomposition, sessions/forking |
| [D2 Tool Design & MCP Integration](domains/d2-tool-design-mcp.md) | 18% | Tool descriptions, structured errors (isError), tool_choice & distribution, .mcp.json scoping, built-in tools |
| [D3 Claude Code Config & Workflows](domains/d3-claude-code.md) | 20% | CLAUDE.md hierarchy, commands & skills, path rules, plan mode, refinement, CI/CD (-p, --output-format json) |
| [D4 Prompt Engineering & Structured Output](domains/d4-prompting-structured-output.md) | 20% | Explicit criteria, few-shot, tool_use + JSON schemas, validation-retry, Batches API, multi-pass review |
| [D5 Context Management & Reliability](domains/d5-context-reliability.md) | 15% | Long-context hygiene, escalation, error propagation, codebase exploration, confidence calibration, provenance |

## Scenario bank

Each sitting draws **4 of 6**, and every file has diagrams + sample questions:

1. Customer Support Resolution Agent ([link](scenarios/s1-customer-support-agent.md))
2. Code Generation with Claude Code ([link](scenarios/s2-code-generation.md))
3. Multi-Agent Research System ([link](scenarios/s3-multi-agent-research.md))
4. Developer Productivity with Claude ([link](scenarios/s4-developer-productivity.md))
5. Claude Code for CI/CD ([link](scenarios/s5-ci-cd.md))
6. Structured Data Extraction ([link](scenarios/s6-structured-data-extraction.md))

See which scenarios test which domains ([here](scenarios/README.md)).

## Official docs coverage

[**official-docs-coverage-map.md**](official-docs-coverage-map.md) maps the exam-relevant pages of [code.claude.com/docs](https://code.claude.com/docs) onto these domain chapters — what's distilled, what's a remaining gap, and what's deliberately out of scope for CCA-F.
