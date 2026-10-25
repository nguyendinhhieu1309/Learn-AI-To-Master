# Agent Evaluation, Security, and Operations

## Learning objectives

Create an evaluation set for multi-step systems, inspect traces, and protect users and tools from common failure paths.

## 1. Evaluate tasks, not demonstrations

Write test cases that include normal requests, ambiguous requests, missing information, unavailable tools, malformed tool output, and adversarial input. Define a successful outcome before looking at model answers. Run the same suite after prompt, model, tool, or retrieval changes.

Track task success, factual correctness, tool-call correctness, invalid actions, refusals/abstentions, cost, latency, and recovery after errors. A single average hides important failures; report categories and severity.

## 2. Inspect traces

A trace records model steps, tool requests, tool responses, timing, and final answer. Use traces to locate whether failure came from misunderstanding, retrieval, a tool, state handling, or evaluation. Store only what is needed, redact sensitive values, and control access and retention.

## 3. Threats and defenses

- **Prompt injection:** instructions in a user message, web page, document, or tool result try to override system policy. Treat external content as data, isolate it clearly, and enforce permission checks outside the model.
- **Excessive agency:** an agent can take broad or irreversible actions. Use least privilege, read-only defaults, explicit human confirmation for consequential actions, and reversible operations where possible.
- **Data exposure:** prompts and traces may contain personal or confidential information. Minimize collection, redact, restrict access, and set retention limits.
- **Tool misuse:** validate inputs, rate-limit, scope queries, and check outputs. Do not rely on the model to enforce access control.
- **Hallucinated completion:** require evidence of tool success; distinguish completed, partial, and failed outcomes.

## 4. Human review and escalation

Set clear boundaries for when the system should ask a human: uncertain identity/permissions, conflicting evidence, high-impact decisions, or irreversible actions. Human review should be an explicit part of the workflow, not an unexplained fallback after failures.

## 5. Operational controls

Set timeouts, max steps, cost ceilings, monitoring alerts, fallback behavior, and a kill switch. Version the prompt, model, tools, and knowledge index. Roll out changes gradually and keep a rollback path.

## Practice

Write 12 evaluation cases for a document assistant, including an injected instruction inside a document and an unavailable search service. For each case, write expected tool behavior and safe final behavior. Review the traces rather than only the final text.

## Common mistakes

- Testing only happy-path prompts written by the system author.
- Using an LLM judge without human review or calibration.
- Believing delimiters alone prevent prompt injection.
- Logging every prompt and result forever because it might be useful later.
