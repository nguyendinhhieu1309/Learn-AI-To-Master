# Agentic Patterns and Human Control

An agent pattern is a control-flow choice. ReAct, planning, reflection, routers, and multi-agent handoffs each trade flexibility for latency, cost, and reliability. Select the smallest pattern that satisfies the task contract.

## Learning objectives

- Describe ReAct, plan-and-execute, reflection, routing, and delegation.
- Decide when parallel tools or specialized agents help.
- Represent approval and pause/resume as explicit state transitions.
- Prevent repeated or stale approvals from triggering duplicate side effects.
- Evaluate complete trajectories rather than only final text.

## Prerequisites

Complete [agent workflows](01-agent-workflows-and-tools.md), [LangChain and LangGraph](05-langchain-and-langgraph.md), [multi-agent systems](07-multi-agent-systems.md), and [context, memory, and persistence](09-context-engineering-memory-and-persistence.md).

## 1. ReAct: alternate tool use and observation

ReAct alternates between selecting an action, observing its result, and selecting the next action. A practical trace can record:

1. current subgoal;
2. proposed tool and validated arguments;
3. tool result or failure;
4. evidence learned;
5. next action or stop reason.

This structured trace is enough to review behavior. Do not require or expose private chain-of-thought; record concise, auditable reasons, evidence, and actions instead.

Use ReAct when the next tool choice depends on what the previous tool returned. Add a step budget, deadline, allowed-tool set, and explicit finish condition.

## 2. Plan and execute

A planner first decomposes a task into steps; an executor carries them out. This can improve coverage for long tasks and make progress visible, but plans can be wrong or become stale when observations change.

A robust planner should:

- produce bounded, typed steps;
- identify dependencies and expected evidence;
- mark steps that require approval;
- allow replanning when evidence contradicts an assumption;
- stop when the task is complete or a safe continuation is impossible.

Do not let a generated plan directly authorize a side effect. Every action still goes through policy and tool validation.

## 3. Reflection and critique

A reflection step checks a draft or trajectory against explicit criteria: completeness, evidence support, schema validity, safety, or task constraints. A separate validator can be deterministic for rules that can be checked deterministically.

Reflection helps only if:

- the critique rubric is concrete;
- the system can identify which issue to fix;
- retries are bounded;
- the evaluator is not simply repeating the same model's unsupported confidence.

Use a hard stop after a fixed number of revisions and return remaining uncertainty instead of looping indefinitely.

## 4. Routing and specialized experts

A router selects a specialized workflow or tool based on the request. Specialization can improve focus, but requires reliable routing and clear responsibility for synthesis.

Parallel expert calls are useful when:

- subtasks are independent;
- each expert has a narrow role and evidence scope;
- the final synthesis can preserve disagreements and provenance.

Sequential delegation is useful when a later step depends on an earlier result. Parallelism may reduce wall time but can increase token cost, duplicated retrieval, and the chance of conflicting answers.

Do not create a separate agent for every tool. A deterministic function is cheaper and easier to test when the step is known.

## 5. Human-in-the-loop state machine

Represent a consequential action as explicit states:

| State | Meaning | Allowed next step |
| --- | --- | --- |
| `drafted` | proposed action is prepared | validate, edit, or reject |
| `pending_approval` | waiting for authorized reviewer | approve, edit, reject, or expire |
| `approved` | reviewer approved a specific action version | revalidate, then execute once |
| `executed` | side effect completed and recorded | report result |
| `rejected` / `expired` | action must not run | stop or create a new proposal |

Approval applies to a specific action payload. If the payload changes after approval, require a new approval. Record an action ID and idempotency key.

## 6. Checkpoint, resume, replay, and edit

A checkpoint should capture the minimum state needed to continue safely. On resume:

1. load state from the correct user/thread;
2. verify authorization and current policy;
3. check whether the approval is still valid;
4. determine whether the external action already happened;
5. execute at most once using idempotency;
6. record the resulting state and trace.

Replay is useful for debugging but can repeat tool calls if side effects are not isolated. Use dry-run tools or recorded tool outputs during replay. Editing state can help recover a stuck process, but require an audit record and re-run invariants after edits.

## 7. Choose a pattern

| Task property | Starting pattern |
| --- | --- |
| Fixed sequence, no uncertain tool choice | deterministic workflow |
| Tool choice depends on observations | bounded ReAct loop |
| Many dependent subtasks | plan-and-execute with replanning |
| Draft needs independent checks | generation followed by validator/critic |
| Independent expert analyses | parallel specialized calls plus synthesis |
| Irreversible action or high impact | explicit human approval state |

Patterns can be combined, but every added loop or delegation boundary creates a new failure and evaluation surface.

## 8. Evaluate the trajectory

For each run, inspect:

- whether the task completed;
- whether each tool call was necessary and authorized;
- whether observations changed the next action appropriately;
- whether evidence supports the final response;
- whether the stop condition worked;
- whether retries, delegation, and review fit the budget;
- whether the final action matched the approved payload.

Success at the final-answer level can hide an unsafe path. Keep trajectory-level checks for tools and approvals.

## Practice with answer checks

1. A plan says to email a customer after lookup. What additional condition is needed before sending?
2. When is parallel expert analysis a poor choice?
3. Why should an approval bind to a specific payload?
4. How can replay duplicate an external action?
5. What should happen when the reflection step reaches its retry limit?

### Answers

1. Permission and policy checks, a valid approval if required, fresh recipient and content validation, and idempotent execution.
2. When subtasks depend on each other's results, cost is constrained, or conflicting outputs cannot be reconciled reliably.
3. Otherwise a different or modified action could run under an approval for an earlier proposal.
4. Replayed tool nodes may re-send a message or repeat a transaction unless side effects are mocked or idempotent.
5. Stop and return the unresolved issue or request human review; do not continue an unbounded loop.

## Mini-project

Build a small research workflow that retrieves evidence, asks two independent specialists to analyze separate questions, synthesizes disagreements, and pauses before any external side effect. Add an explicit state diagram, maximum-step policy, checkpoint/resume test plan, idempotency strategy, and trajectory evaluation rubric.

## Common mistakes

- Treating a plausible plan as authorization to execute it.
- Asking a model to critique itself without an independent rubric.
- Running parallel agents whose tasks overlap and conflict.
- Resuming an old approval after the action payload changed.
- Replaying production traces against live side-effecting tools.

## References

- [Thinking in LangGraph](https://docs.langchain.com/oss/javascript/langgraph/thinking-in-langgraph) — explicit nodes, state, errors, interrupts, and persistence.
- [LangChain learning resources](https://docs.langchain.com/oss/python/learn) — workflow, skills, and multi-agent tutorials.
