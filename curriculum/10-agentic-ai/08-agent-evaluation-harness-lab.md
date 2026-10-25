# Agent Evaluation Harness Lab

An agent demonstration shows one successful trajectory. An evaluation harness asks whether the system succeeds reliably across a defined set of tasks, uses tools appropriately, stops safely, and remains observable when dependencies fail.

## Learning objectives

- Write measurable agent task and safety contracts.
- Build an evaluation set that includes normal, adversarial, and failure cases.
- Separate task quality, tool correctness, safety, latency, and cost.
- Trace a run without logging secrets or unnecessary personal data.
- Choose when a fixed graph is safer than an open-ended agent loop.

## Prerequisites

Study [agent workflows and tools](01-agent-workflows-and-tools.md), [evaluation and operations](02-evaluation-security-and-operations.md), [skills](03-skills-and-reusable-procedures.md), [harness design](04-harness-design-and-runtime.md), [LangChain and LangGraph](05-langchain-and-langgraph.md), and [MCP](06-mcp-and-interoperability.md).

## 1. Specify the contract before the prompt

Example task: answer questions about a small internal handbook using approved search and read tools.

**Success conditions**

- answer is supported by retrieved passages;
- every factual claim has a source reference;
- the system says when evidence is missing;
- tool arguments satisfy the declared schema;
- the agent stops within a fixed step and time budget.

**Safety conditions**

- no write-capable tools are available;
- retrieved text is treated as untrusted data, not as new system instructions;
- the agent cannot reveal content outside the user's authorization;
- tool errors do not cause invented citations.

These conditions turn “helpful agent” into observable assertions.

## 2. Define the harness boundary

A maintainable harness separates:

1. **Task input:** normalized user request and authorization context.
2. **Policy:** allowed tools, budget, approval rules, and stop conditions.
3. **Model adapter:** prompt assembly, structured output parsing, retry limits.
4. **Tool adapter:** argument validation, permission check, timeout, result normalization.
5. **State:** task status, evidence references, tool results, and retry count.
6. **Observer:** trace events, metrics, redaction, and final result.
7. **Evaluator:** fixed cases, expected invariants, and human review.

The model may propose a tool call. The harness decides whether it is allowed and whether it executes.

## 3. Use a bounded control loop

Framework-neutral pseudocode:

~~~text
state = initialize(task, authorization, budget)

while not state.finished:
    if state.steps >= budget.max_steps:
        return safe_stop("step_budget_exhausted")
    if clock() >= budget.deadline:
        return safe_stop("deadline_exceeded")

    proposal = model(state.messages, allowed_tool_schemas(state))
    decision = policy.validate(proposal, state.authorization, state)

    if decision.requires_human_approval:
        state = pause_and_record(proposal, decision.reason)
        return state
    if decision.is_denied:
        state = append_denial(state, decision.reason)
        continue_or_stop_according_to_policy(state)
    if proposal.is_final:
        return validate_final_answer(proposal, state.evidence)

    result = execute_with_timeout_and_idempotency(proposal)
    state = append_tool_result_and_trace(state, result)
~~~

The loop requires a maximum number of steps, a deadline, a tool policy, and an explicit final-answer validation rule. Retries must be bounded too; an agent that loops forever is a product defect even if every individual call is valid.

## 4. Design tools and skills as controlled capabilities

A tool should have:

- a narrow purpose and typed arguments;
- input limits and validation;
- an authorization check independent of the prompt;
- a timeout and clear error contract;
- an idempotency strategy for side effects;
- output limits and sensitive-data redaction.

A skill is a reusable procedure that can explain when to use a tool and how to verify the result. Keep the skill separate from permission enforcement: text instructions can guide the model, but the tool adapter must still reject unauthorized calls.

## 5. Build a small but adversarial evaluation set

Create cases in categories:

| Category | Example |
| --- | --- |
| Direct answer | question answered by one known section |
| Multi-step retrieval | answer requires comparing two sections |
| No evidence | answer absent from the handbook |
| Conflicting evidence | two dated policies disagree |
| Injection attempt | retrieved content asks the agent to ignore its policy |
| Invalid tool call | unknown tool or malformed argument |
| Tool outage | search times out or returns an error |
| Authorization boundary | user requests a document they cannot access |
| Budget boundary | question induces repeated searches |

Each case should include expected behavior, not necessarily one exact sentence. For example, an unanswerable case should require abstention and prohibit fabricated sources.

## 6. Measure dimensions separately

Track:

- **task success:** answer meets the contract;
- **grounding:** claims are supported by evidence;
- **tool correctness:** allowed tool and arguments were used;
- **safety:** prohibited action or disclosure did not occur;
- **efficiency:** number of model and tool calls;
- **reliability:** success across repetitions and dependency failures;
- **latency and cost:** end-to-end and per-stage values.

Do not collapse all of these into one score too early. A weighted score can hide a catastrophic safety failure behind strong answer quality. Safety invariants should often be pass/fail gates.

## 7. Trace the run without over-logging

Useful trace fields include:

- run and task identifiers;
- model, prompt, tool, skill, and policy versions;
- step number and stop reason;
- tool name, validated argument summary, status, and latency;
- evidence identifiers rather than copied sensitive passages where possible;
- token or cost estimate and retry count.

Do not store secrets, access tokens, or full user content by default. Define retention, access control, redaction, and deletion behavior before turning on traces.

## 8. LangChain, LangGraph, and execution choice

Use a higher-level LangChain agent when the task is a relatively simple tool-using loop and its defaults match the required behavior. Use a LangGraph workflow when the product needs explicit state transitions, durable checkpoints, branching, retries, or human pauses. A graph can make a workflow inspectable; it does not by itself make the policy secure or the evaluation valid.

Keep business rules and authorization outside framework-specific prompt text. Pin and test the framework version in a real project, and verify that checkpoint storage does not retain sensitive data longer than intended.

## 9. Failure-injection lab

Run the evaluation suite with one failure at a time:

1. search tool returns no results;
2. tool call exceeds its deadline;
3. model returns malformed structured output;
4. the same tool call is proposed repeatedly;
5. a user denies a requested approval;
6. the agent is interrupted and resumed from a checkpoint;
7. retrieved content contains adversarial instructions.

For each case, record the expected stop or recovery path, actual trace, user-visible message, and whether the agent used an unauthorized capability.

## Practice with answer checks

1. Why should a tool adapter check authorization even if the system prompt says “never access private documents”?
2. Why is a step budget part of correctness?
3. How can an evaluation set overfit the prompt?
4. When is a fixed state graph preferable to an open-ended loop?
5. Why can logging every tool result create a security problem?

### Answers

1. The prompt guides model behavior but is not an enforcement boundary. The adapter has the actual capability and must check permission at execution time.
2. A task that never terminates can consume unbounded time and money and fail its service contract.
3. If the prompt is tuned repeatedly to a small visible set, results may reflect memorization of those cases. Keep a hidden or newly authored regression set.
4. When the steps, transitions, approval points, or side effects are known and should be inspectable.
5. Results can contain personal data, secrets, retrieved private text, or adversarial instructions; traces require minimization and access controls.

## Mini-project

Build a read-only handbook agent with a reusable search skill, two narrow tools, a fixed evaluation set of at least 25 cases, explicit abstention behavior, a bounded loop, structured traces, and a failure-injection report. Include at least five safety or authorization cases. Do not add write tools to this project.

## Common mistakes

- Evaluating only happy-path prompts.
- Treating a model's claimed citation as proof that the citation supports the answer.
- Treating tool schemas as authorization.
- Retrying every failure without a deadline or retry cap.
- Letting framework defaults silently control permissions or checkpoint retention.

## References

- [LangChain learning resources](https://docs.langchain.com/oss/python/learn) — tutorials for agents, skills, and LangGraph workflows.
- [Thinking in LangGraph](https://docs.langchain.com/oss/javascript/langgraph/thinking-in-langgraph) — decomposing workflows into stateful nodes, errors, checkpoints, and human review.
