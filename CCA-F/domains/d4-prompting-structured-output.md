# F-D4 · Prompt Engineering & Structured Output (20%)

Precision prompting (explicit criteria, few-shot), guaranteed-schema output, validation-retry loops, batch processing, and review architectures.

**Tested by scenarios:** [⑤ CI/CD](../scenarios/s5-ci-cd.md) · [⑥ Structured Data Extraction](../scenarios/s6-structured-data-extraction.md)
**Source:** official [CCA-F Exam Guide](../../official-exam-guides/cca-f-exam-guide.pdf), task statements 4.1–4.6.

---

## The shape of this domain

This domain covers two different problems. Each needs a different solution.

- **Is the output the right shape?** That is mechanical, and you can guarantee it (4.3).
- **Is the output right?** That is judgment, and no schema can enforce it. It comes from criteria, examples, validation and review architecture (4.1, 4.2, 4.4, 4.6).

Do not use the first answer for the second problem. An extraction can match its schema and still be wrong, such as when its line items do not match the stated total.

**How the sections connect.** Sections 4.1 and 4.2 replace vague requests with clear criteria and examples. Section 4.3 guarantees the output *shape*, while 4.4 validates what the schema cannot check. Section 4.5 covers large workloads, and 4.6 covers review. Section 4.7 explains where these instructions belong when using the SDK.

## 4.1 Explicit criteria over vague instructions

**The problem:** A review agent reports too many issues. You tell it to be more careful.

**The cause:** Instructions such as "be conservative" and "only report high-confidence findings" do not define a threshold. The model must create one, and it may change between runs.

**The solution:** Replace vague descriptions with a test the model can apply.

- ✅ "Flag comments only when claimed behavior contradicts actual code behavior."
- ❌ "Check that comments are accurate."
- **For consistent severity levels, give clear criteria and one code example per level.** A definition alone leaves the boundary open to interpretation.
- **One noisy category reduces trust in all results.** Disable it while improving its prompt so developers do not learn to ignore the whole tool.
- **Use a prompt-based self-audit when the work needs more complete rationale, not human oversight.** Require the agent to check that its response resolves every issue, includes relevant context and timing, and anticipates a follow-up question before it answers.

## 4.2 Few-shot prompting
([Multishot prompting](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/multishot-prompting))

**The problem:** Your criteria are clear, but the output format still changes. The criteria define *what* to report, not *how* to report it.

**The solution:** Show the model examples. Instructions describe the result; examples demonstrate it. This is one of the best ways to get consistent, useful output.

- **Use examples for unclear cases**, not obvious ones. Two to four examples usually work well.
- **Show the reasoning**, not only the answer. Explain why one action was better than a reasonable alternative. This teaches a rule instead of a simple pattern.
- **Good examples apply to new cases.** They also reduce invented details when documents use different structures or informal wording.
- **Show the required output shape**, such as location, issue, severity, and suggested fix.

## 4.3 Structured output
([Structured outputs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs))

**The problem:** Examples make the format consistent enough for people, but another system must now parse it. Asking for JSON in the prompt works only most of the time.

**The cause:** A parser needs a guarantee. Occasional syntax errors, missing fields, or changing types create unnecessary error handling. Examples make valid output likely, not certain.

**The solution:** Use **constrained decoding**. While Claude generates the response, the API allows only tokens that keep it valid against the schema. Invalid output is prevented, not generated and retried. Two features use this method, separately or together.

```mermaid
flowchart TD
    G["Need machine-parseable output"] --> L1["📝 'Return JSON' in the prompt: syntax errors possible"]
    G --> L2["🛠️ output_config.format: type: json_schema — constrained decoding"]
    G --> L3["🔒 strict: true on a tool: guarantees tool NAME + INPUT validity"]
    L2 --> BUT["⚠️ Guarantees SHAPE, not TRUTH: line items ≠ total, values in the wrong fields"]
    L3 --> BUT
    classDef weak fill:#90A4AE,color:#000,stroke:#455A64,stroke-width:2px
    classDef strong fill:#E64A19,color:#fff,stroke:#BF360C,stroke-width:2px
    classDef q fill:#455A64,color:#fff,stroke:#263238,stroke-width:2px
    class L1 weak
    class L2,L3,BUT strong
    class G q
```

- **JSON outputs:** Set `output_config.format` to `{ "type": "json_schema", "schema": {…} }`. The response is guaranteed to match. This is generally available for Claude 4.5 and later. The earlier beta used top-level `output_format` with a `structured-outputs-2025-11-13` header; both forms still work during the transition.
- **Strict tool use:** Set `strict: true` on a tool definition to validate the tool name and inputs against the schema. Use this when the required structure is a tool call.
- **Only part of JSON Schema is supported.** Unsupported features include recursive schemas, external `$ref`, numeric constraints (`minimum`, `maximum`, `multipleOf`), string-length constraints, array constraints beyond `minItems` of 0 or 1, and any `additionalProperties` value except `false`. Unsupported features return a detailed 400 error instead of silently using weaker validation.

### Schema design that avoids fabrication

- **Choose required fields carefully.** Require a field only when it is genuinely always present. Otherwise, make it optional or nullable, because a required field can push the model to invent a value.
- **Allow null for missing values:** For example, `"type": ["string", "null"]` lets the model return `null` instead of fabricating a string.
- **Keep enums open:** Add `"other"` with a free-text detail field so a new category is not silently dropped.
- **Give uncertain classifications an honest fallback:** Use `"unclear"` when the model cannot classify confidently. This is better than a confident wrong label.
- **Add format-normalization rules to the prompt** when sources are inconsistent. The schema controls the shape; the prompt explains how to normalize the data.

**The limit:** Constrained decoding guarantees valid, schema-matching output. It does not guarantee correct values. Section 4.4 covers that problem.

## 4.4 Validation, retry & feedback loops

**The problem:** The extraction matches its schema, but its line items do not match the stated total.

**The cause:** Constrained decoding checks only the output *shape*. It does not check arithmetic or consistency between fields. Add a validation step outside the model.

The validator can be a Pydantic model in Python or a JSON Schema validator in another language. The specific tool matters less than running it before trusting the output.

```mermaid
sequenceDiagram
    participant P as Pipeline
    participant C as Claude
    participant V as Validator (Pydantic / JSON schema)
    P->>C: document + extraction schema
    C-->>P: extraction
    P->>V: validate
    V--xP: ❌ semantic error: line items ≠ total
    P->>C: retry: document + failed extraction + SPECIFIC validation error
    C-->>P: corrected extraction
    Note over P: Retry helps format/structure errors. It cannot conjure info absent from the source.
```

- **Retry with the exact error.** Send the document, failed extraction, and validation message. "That was wrong, try again" does not explain what to fix.
- **Retries can fix format and structure, but cannot create missing information.** If a field is absent from the document, make it nullable (4.3) instead of retrying.
- **Put checks in the schema when possible.** Extract `calculated_total` beside `stated_total` to show differences directly. Use a `conflict_detected` boolean for inconsistent sources.
- **Add `detected_pattern` to each finding** so dismissed findings become useful data. Over time, this creates a clear list of false-positive patterns.
- **Use reasoning fields for validation feedback, not only auditability.** A `detected_patterns` field lets the validator compare the evidence with the category and return a targeted inconsistency for self-correction.

## 4.5 Batch processing (Message Batches API)
([Batch processing](https://platform.claude.com/docs/en/build-with-claude/batch-processing))

**The problem:** You need to process 40,000 documents, while a separate pre-merge check must return quickly to a waiting developer.

**The solution:** Use different APIs for different workloads. The **Message Batches API** processes many Messages requests asynchronously at half the standard price. Use it when immediate results are not required.

```mermaid
flowchart TD
    W{"Workflow latency tolerance?"} -- "blocking (pre-merge check)" --> RT["⚡ Real-time API"]
    W -- "latency-tolerant (overnight reports, audits, nightly test generation)" --> B["📦 Batches API: 50% cost savings"]
    B --> F["Most finish <1h · hard 24h expiry"]
    B --> CID["custom_id correlates request↔response"]
    F --> RESUB["Failures: resubmit ONLY failed docs (by custom_id), with fixes, e.g. chunk docs that exceeded context"]
    classDef rt fill:#90A4AE,color:#000,stroke:#455A64,stroke-width:2px
    classDef batch fill:#E64A19,color:#fff,stroke:#BF360C,stroke-width:2px
    classDef q fill:#455A64,color:#fff,stroke:#263238,stroke-width:2px
    class RT rt
    class B,F,CID,RESUB batch
    class W q
```

- **Batch usage costs 50% of standard prices.**
- **There is no latency SLA.** Most batches finish within an hour, but unfinished batches **expire after 24 hours**. Plan for the 24-hour limit, not the common one-hour result. For a 30-hour downstream SLA, submit work in roughly four-hour windows.
- **`custom_id` connects each response to its request.** It must be 1–64 characters and match `^[a-zA-Z0-9_-]{1,64}$`. Results may arrive out of order, so every request needs one.
- **A batch caps at 100,000 requests or 256 MB**, whichever comes first.
- **Tools work in batches**, including server tools such as web search and code execution. Unsupported parameters are `stream`, `speed`, `store`, `previous_thread_event_id`, `cache_hint`, `context_hint`, `max_tokens: 0`, and `research_preview_2026_02`.
- **Test the prompt on a sample first.** Finding a prompt problem after processing 40,000 documents wastes the full run.

## 4.6 Multi-instance & multi-pass review

**The problem:** A validator can check arithmetic and field consistency, but some quality checks need judgment. You ask the same session that wrote the code to review it.

**The cause:** The session still holds the reasoning that produced the code. It is likely to repeat the same justification instead of questioning it. A separate instance reviews more effectively than the original session, even with more thinking time.

```mermaid
flowchart LR
    GEN["Session that wrote the code"] -. "retains its own reasoning, won't question its decisions" .-> SELF["❌ self-review"]
    IND["🔎 Independent instance (no prior reasoning context)"] --> BETTER["✅ catches subtle issues"]
    BIG["14-file PR, one pass"] --> DILUTE["❌ attention dilution: inconsistent depth, contradictions"]
    SPLIT["Per-file local passes + cross-file integration pass"] --> BETTER
    classDef bad fill:#B71C1C,color:#fff,stroke:#7F0000,stroke-width:2px
    classDef good fill:#E64A19,color:#fff,stroke:#BF360C,stroke-width:2px
    classDef n fill:#455A64,color:#fff,stroke:#263238,stroke-width:2px
    class SELF,DILUTE bad
    class IND,SPLIT,BETTER good
    class GEN,BIG n
```

- **Review in a separate instance** without the original reasoning context.
- **Split large reviews into one pass per file and a separate integration pass.** This applies [F-D1 §6.3](d1-agentic-architecture.md) to review. **A larger context window does not improve attention quality**; it only allows more material in one pass.
- **Use confidence reports for routing, not filtering.** Send uncertain findings to a person instead of removing them.

## 4.7 Shaping the system prompt (Agent SDK)
([Modifying system prompts](https://code.claude.com/docs/en/agent-sdk/modifying-system-prompts))

**The problem:** Sections 4.1–4.6 explain *what* to write. When building an agent with the Agent SDK ([F-D1 §1](d1-agentic-architecture.md)), you must also decide *where* those instructions belong.

**Why it matters:** The **system prompt** appears before the conversation and applies to every turn. Claude Code includes tool guidance, safety rules, and coding conventions in its system prompt. With the SDK, you can use, replace, or extend it. Your choice decides which instructions you must provide yourself.

**The deciding question:** How similar is your agent to Claude Code, where a person watches a coding agent in a repository? The less similar it is, the more instructions you should write yourself.

| Starting point | `systemPrompt` value | Keeps Claude Code's tool guidance + safety rules? |
|---|---|---|
| Minimal default | *(unset)* | No - tool-calling support only |
| `claude_code` preset | `{ type: "preset", preset: "claude_code" }` | Yes - the full CLI prompt |
| Preset **+ `append`** | `…preset, append: "…"` | Yes, plus your additions - **lowest-risk customization** |
| Custom string | `"You are…"` | No - you re-add any safety and tool guidance yourself |

- **CLAUDE.md is not the system prompt.** The SDK adds it to the **conversation** when `settingSources` / `setting_sources` includes `project` or `user`. It works with any system prompt and does not change the cache-sensitive system prefix.
- **Output styles** (`.claude/output-styles/*.md` and `outputStyle`) **replace** the preset's software-engineering instructions unless `keep-coding-instructions: true` is set. Built-in styles are Default, Proactive, Explanatory, and Learning. They work in the CLI and SDK, and changes apply after `/clear` or a restart.
- **Caching:** The preset places session-specific context such as cwd, Git state, OS, and shell before your `append`. Sessions from different directories therefore miss the cache. `excludeDynamicSections: true` moves this context to the first user message, allowing machines to share one cached system prefix. The CLI flag is `--exclude-dynamic-system-prompt-sections`. See [F-D5 §5.8](d5-context-reliability.md).

**Routing clue:** A systematic keyword-driven pattern usually signals explicit routing logic in the system prompt. Do not add a pre-processing layer unless it has a separate, necessary purpose.

**Study prompts:** Revisit Q39 and Q42 in the [external guide](https://github.com/paullarionov/claude-certified-architect/blob/main/guide_en.md).

> **Sources:** [Structured outputs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs) covers `output_config.format`, `strict`, and the supported JSON Schema features. [Batch processing](https://platform.claude.com/docs/en/build-with-claude/batch-processing) covers pricing, the 24-hour expiry, `custom_id`, and batch limits. [Multishot prompting](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/multishot-prompting) covers example-based prompting. [Modifying system prompts](https://code.claude.com/docs/en/agent-sdk/modifying-system-prompts) covers presets, output styles, and `excludeDynamicSections`.

---

## Exam traps checklist

| Trap | Correct instinct |
|---|---|
| "Be more careful" style instructions | Explicit categorical criteria |
| Required schema fields for possibly-absent data | Nullable/optional fields (else fabrication) |
| "Schema-valid means correct" | Constrained decoding guarantees shape, never truth - validate semantics (4.4) |
| Reaching for tool_use as the only way to get JSON | `output_config.format` is the direct route; `strict: true` covers tool inputs |
| Retry until absent data appears | Retry only fixes format/structure errors |
| Batch API for blocking pre-merge checks | Real-time for blocking; batch for overnight |
| "Batch results can't be matched to requests" | `custom_id` exists for exactly that |
| "Batches can't use tools" | Tool use works, including server tools |
| Planning against "most batches finish in an hour" | 24h is the hard expiry - plan against that |
| Bigger model/context to fix diluted review | Multi-pass architecture |
| Consensus voting across runs to cut false positives | Suppresses real intermittent findings (sample Q12, option D) |

**Practice:** [claude-cookbooks `evals` + extraction recipes](https://github.com/anthropics/claude-cookbooks) · [anthropics/courses prompt-evaluations](https://github.com/anthropics/courses) · Exercise 3 in the official guide (full extraction pipeline).


## TL;DR


- **Prompt clearly and directly**
    - Be explicit about the task, desired output, format, and constraints.
    - If order/completeness matters, specify steps using numbered lists or bullets.
    - Don't expect Claude to infer "go above and beyond"; explicitly ask for it.

- Add context / motivation = explain WHY an instruction matters; Claude can generalize better from the reason.

- **Few-shot / multishot examples** = one of the most reliable ways to control output, tone, format, and consistency.
    - Prefer 3–5 examples.
    - Examples should be relevant + diverse + representative of edge cases.
    - Wrap examples in `<example>` / `<examples>` tags.

- **XML tags** = separate different parts of complex prompts clearly.
    - eg. `<instructions>`, `<context>`, `<examples>`, `<input>`, `<documents>`
    - Use consistent descriptive tags and nest them when there is a natural hierarchy.

- Role prompting = give Claude a role in the system prompt to focus behaviour/tone.
    - eg. `You are a coding assistant specializing in Python.`

- **Long-context prompting**
    - Put long documents/data near the TOP of the prompt.
    - Put the actual query/instructions AFTER the documents.
    - For multiple documents, use XML tags with document content + source metadata.
    - For grounding, ask Claude to extract relevant quotes first, then perform the task.

- **Output formatting**
    - Tell Claude what TO do instead of only what NOT to do.
    - Match the style of your prompt to the style you want in the output.
    - For strict machine-readable output, prefer Structured Outputs / schemas rather than tricks such as response prefilling.

- Prefilled assistant responses are no longer supported on newer Claude models; use explicit instructions, XML, Structured Outputs, or tool calling instead.

- **Thinking / reasoning**
    - Adaptive thinking = Claude dynamically decides how much reasoning is required.
    - `effort` controls how much thinking Claude uses; higher effort generally means more reasoning/cost/latency.

- **Long-running / agentic tasks**
    - Persist important state across context windows using structured files, progress notes, or git.
    - Git = useful for checkpoints + tracking state across sessions.
    - Give the agent verification tools so it can validate work without continuous human feedback.

- **Autonomy vs safety**
    - Encourage autonomous local/reversible actions such as editing files and running tests.
    - Require confirmation for destructive, hard-to-reverse, or externally visible actions.
    - eg. deleting data, force-push, modifying shared infrastructure, posting externally.

- **Subagents**
    - Use subagents for parallel work, isolated context, or independent workstreams.
    - Avoid subagents for simple tasks, single-file edits, or highly sequential work requiring shared context.

- Prompt chaining = useful when you need explicit intermediate checkpoints.
    - Common pattern: `draft → review against criteria → refine`
    - Use separate calls when intermediate results need to be inspected, logged, evaluated, or branched.

-----

- **Structured Outputs** = constrain Claude to return schema-compliant, parseable output instead of relying only on prompting.
    - Avoids malformed JSON, missing required fields, wrong data types, and schema violations.

- **JSON outputs** = control WHAT Claude returns.
    - Define a JSON Schema using `output_config.format`  (in `client.messages.create(...)` when making API call)
    - `type: "json_schema"` + schema definition.
    - Response is valid JSON matching the schema.
    - Good for extraction, classification, structured reports, and API responses.

- **Strict tool use** = control HOW Claude calls tools.
    - Set `strict: true` on a tool.
    - Guarantees the tool name/input follows its JSON Schema.

- **Schema compliance exceptions**
    - `stop_reason: "refusal"` → safety refusal may not follow your schema.
    - `stop_reason: "max_tokens"` → truncated output may not follow your schema.
    - Always inspect `stop_reason` before assuming the structured output is valid.

- Keep schemas reasonably simple.
    - Too many optional fields, unions, nested objects, or strict tools increase grammar complexity.
    - If too complex → simplify schema, reduce strict tools, or split work across requests.

- Structured Outputs work with streaming and batch processing.
- JSON Structured Outputs are incompatible with citations and message prefilling.

- Key distinction:
    - Prompting for JSON = Claude tries to follow the format.
    - Structured Outputs = API constrains Claude to the schema.

----


- **Batch processing** = process many Messages API requests asynchronously instead of waiting for immediate responses.
    - Best for large-volume, non-interactive workloads such as evaluations, moderation, analysis, and bulk generation.
    - Each request in the batch is processed independently.
    - 50% lower API cost than normal synchronous requests.
    - Most batches finish within 1 hour.

- **Batch lifecycle**
    - Submit batch → status starts as `in_progress`
    - Poll/check status → becomes `ended` when processing finishes
    - Retrieve results after completion
    - Batch may also be canceled while processing

- Each request needs a unique `custom_id`.
    - Use `custom_id` to match results back to inputs.
    - Results are NOT guaranteed to return in the same order as requests.

- **Per-request result states**
    - `succeeded` = request completed successfully
    - `errored` = request failed
    - `canceled` = request was canceled before processing
    - `expired` = request was not processed before batch expiration

- **Limits**
    - Maximum `100,000` requests OR `256 MB` per batch, whichever comes first.
    - Batch expires after `24 hours` if processing has not completed.
    - Results remain downloadable for `29 days`.

- Almost anything supported by the Messages API can be batched, including vision, tool use, system prompts, multi-turn conversations, and extended thinking.

- `stream: true` is NOT supported because batch results are returned after asynchronous processing.
- Batch results are returned as `.jsonl`, one JSON object per request.

- Test request structure with the normal Messages API first because batch validation errors may only appear after batch processing finishes.

- **Prompt caching + batches**
    - Prompt caching can be combined with batch processing.
    - Their cost savings can stack.
    - Cache hits are best-effort because batch requests execute asynchronously/concurrently.
    - For shared prompts in longer-running batches, consider the 1-hour cache duration.

- Key distinction:
    - Messages API = synchronous / immediate response
    - Message Batches API = asynchronous / high-throughput / cheaper


-----


- **Agent SDK system prompt starting points**
    - No `systemPrompt` → minimal default with basic tool-calling support only.
    - `claude_code` preset → full Claude Code prompt with tool guidance, safety/security instructions, and environment context.
    - Custom `systemPrompt` → replaces the default entirely; you must provide any tool/safety/environment instructions yourself.

- Use `claude_code` preset when building a coding agent similar to Claude Code.
- Use `claude_code` + `append` when you want Claude Code behaviour plus extra product/session-specific instructions.
- Use a custom system prompt when the agent has a different identity, surface, permission model, or is not mainly a coding agent.

- `claude -p` uses the Claude Code system prompt by default, but Agent SDK without `systemPrompt` uses the minimal default.

- **CLAUDE.md vs system prompt**
    - CLAUDE.md does NOT modify the system prompt.
    - SDK injects CLAUDE.md into the conversation as project context.
    - CLAUDE.md loading depends on `settingSources`, not on the `claude_code` preset.

- **Output styles** = reusable saved instructions for role, tone, and output format.
    - Can be user-level or project-level.
    - `keep-coding-instructions: true` preserves Claude Code coding instructions when applying a custom output style.

- `append` = safest way to customize Claude Code because it adds instructions without removing built-in behaviour/safety.

- **Prompt caching**
    - Claude Code preset contains dynamic environment data such as working directory, OS, shell, git state, etc.
    - Different environment data can cause prompt-cache misses.
    - `excludeDynamicSections: true` moves dynamic environment context to the first user message so the static system prompt can be cached across machines/sessions.

- **Existing sessions**
    - A resumed/continued session normally keeps the system prompt recorded from its first request.
    - Changing `append` or custom `systemPrompt` may not take effect until compaction or a new session.
    - For mid-session instruction changes, send the new instruction in a user message or inject it through hook `additionalContext`.

- `snapshot: false` rebuilds the system prompt on every request so prompt edits affect resumed sessions immediately.
    - Useful while iterating on prompts.
    - Keep snapshot/recording enabled in production because disabling it reduces cache reuse and session continuity.

- Key distinction:
    - CLAUDE.md = persistent project context
    - Output style = reusable role/tone/output behaviour
    - `append` = add session-specific instructions while preserving Claude Code defaults
    - Custom `systemPrompt` = full control, but built-in safety/tool/environment guidance is lost unless you recreate it

