# Multi-Agent Systems

## Learning objectives

Choose an orchestration pattern, define clean boundaries between agents, and evaluate whether delegation improves results enough to justify its complexity.

## 1. Common patterns

- **Sequential pipeline:** fixed specialists or stages run in order (extract → research → write → verify).
- **Router/handoff:** a triage agent selects one specialist to handle the request.
- **Manager with agents-as-tools:** a manager retains control and calls specialists for bounded subtasks, then combines results.
- **Parallel fan-out/fan-in:** independent tasks run concurrently and a reducer combines results.
- **Collaborative loop:** agents critique or revise one another iteratively; it needs a strict iteration limit and objective rubric.

## 2. Handoff vs. agent-as-tool

With a **handoff**, control transfers to a specialist, which typically becomes the active responder for that branch. With an **agent-as-tool**, the manager calls a specialist for a subtask and remains responsible for the user-facing result. Choose based on who should own the final answer and how much context each specialist needs.

## 3. Boundaries and shared state

Give each specialist a narrow role, explicit input/output schema, and defined success condition. Pass relevant context, not the entire conversation by default. Distinguish shared immutable evidence from mutable task state. Decide how conflicts, missing results, timeouts, and partial completion are handled.

Parallelism is helpful only for independent work. If two agents write to the same resource or depend on each other's conclusions, add coordination or serialize the work. Combine duplicate findings and preserve provenance.

## 4. Cost, latency, and reliability

Each agent may add model calls, tokens, tool executions, and failure points. Measure end-to-end task success, cost, latency, disagreement, and recovery against a single-agent/workflow baseline. Limit recursion and fan-out. A manager should not call specialists repeatedly without a stopping rule.

## 5. Evaluation and security

Test misrouting, conflicting specialist outputs, malicious retrieved content, failed agents, duplicated actions, and permission boundaries. Apply authorization at each tool execution even if the calling specialist is trusted. Keep audit traces that show which agent proposed which action and which result was used.

## Practice

For a technical-support assistant, compare (a) one agent with documentation search, (b) a router with billing and technical specialists, and (c) a manager that asks both specialists for evidence. Specify when each design wins, its failure modes, and how you would measure cost and correct routing.

## Common mistakes

- Adding agents because “multi-agent” sounds more advanced.
- Giving every agent all tools, secrets, and conversation history.
- Allowing uncontrolled delegation loops or unbounded parallel calls.
- Combining conflicting answers without preserving sources or asking for review.
