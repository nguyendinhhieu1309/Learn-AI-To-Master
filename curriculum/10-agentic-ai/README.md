# Agentic AI

Agentic systems use models to choose steps, call tools, or coordinate workflows toward a task. This module treats an agent as a software system with measurable reliability, permissions, and failure modes—not simply a prompt that runs in a loop.

## Outcomes

- Distinguish a fixed workflow from a model-directed agent.
- Design tool interfaces, state, and stopping conditions for a bounded task.
- Evaluate task success, reliability, cost, latency, and safety.

## Prerequisites

Python, API basics, and introductory NLP/language models. Read the [LLM evaluation and safety topics](../08-nlp-llms/README.md) first.

## Suggested sequence

1. [Agent workflows and tools](01-agent-workflows-and-tools.md)
2. [Skills and reusable procedures](03-skills-and-reusable-procedures.md)
3. [Context engineering, memory, and persistence](09-context-engineering-memory-and-persistence.md)
4. [Harness design and runtime](04-harness-design-and-runtime.md)
5. [LangChain and LangGraph](05-langchain-and-langgraph.md)
6. [MCP and interoperability](06-mcp-and-interoperability.md)
7. [Multi-agent systems](07-multi-agent-systems.md)
8. [Agentic patterns and human control](10-agentic-patterns-and-human-control.md)
9. [Evaluation, security, and operations](02-evaluation-security-and-operations.md)
10. [Agent evaluation harness lab](08-agent-evaluation-harness-lab.md)

## Practice

- Compare a deterministic workflow with an agent on the same small task.
- Add input validation, a tool allowlist, a step budget, and an explicit stop condition.
- Test malformed tool output, timeouts, conflicting instructions, and prompt injection in retrieved content.
- Review traces to distinguish model mistakes from tool or data failures.
- Complete the [failure-injection lab](08-agent-evaluation-harness-lab.md) before adding write-capable tools.

## Checkpoint project

Build a bounded assistant that uses a small set of read-only tools. Define its allowed actions, test cases, stopping rule, and evaluation rubric. Report task success, failure categories, cost/latency, and known risks.
