# Context Engineering, Agent Memory, and Persistence

An agent's context is the information available for its next decision. Memory is information retained for later use. These ideas overlap, but they are not interchangeable: saving a conversation does not guarantee the right information will be retrieved, and retrieving a fact does not guarantee it is current or safe to use.

## Learning objectives

- Decompose an agent context window into task, policy, tools, evidence, and state.
- Distinguish working, semantic, episodic, and procedural memory.
- Budget, compress, retrieve, and validate context.
- Separate workflow checkpoints from long-term memory.
- Design memory with provenance, privacy, expiration, and evaluation.
- Use human review and resumable state for consequential workflows.

## Prerequisites

Complete [agent workflows](01-agent-workflows-and-tools.md), [skills](03-skills-and-reusable-procedures.md), [harness design](04-harness-design-and-runtime.md), and [LangChain and LangGraph](05-langchain-and-langgraph.md).

## 1. Context is assembled for a decision

An agent request may include:

- system and developer policy;
- the current user task and authorization context;
- available tool descriptions;
- recent conversation;
- retrieved documents or memories;
- intermediate results and unresolved questions;
- output format and a reserve for the response.

The total input must fit the model's context limit:

$$
T_{\mathrm{policy}}+
T_{\mathrm{task}}+
T_{\mathrm{tools}}+
T_{\mathrm{evidence}}+
T_{\mathrm{history}}+
T_{\mathrm{reserve}}
\leq T_{\mathrm{context}}.
$$

The limit is not a target to fill. More context can raise latency and cost, increase distraction, and make sensitive data exposure more likely.

## 2. Four kinds of memory

| Type | What it stores | Example | Main risk |
| --- | --- | --- | --- |
| Working | current task state and recent interaction | pending approval, current search results | context overflow or stale summaries |
| Semantic | relatively stable facts | preferred language, project glossary | incorrect or outdated fact |
| Episodic | a past event or outcome | a previous support case and resolution | overgeneralizing from one event |
| Procedural | how to perform a task | a skill for validating a CSV | executing an obsolete or unsafe procedure |

Memory should be created because a future task benefits from it, not because storage is available. A system that records everything has made a retention decision, even if nobody designed one.

## 3. Separate workflow state from long-term memory

**Workflow checkpoint:** enough state to resume a particular run at a safe step. It may include current task status, tool results, pending approval, and execution metadata.

**Long-term memory:** a curated fact, episode, or procedure that may be retrieved in future runs.

These have different lifecycles. A checkpoint may expire after a workflow finishes; a user preference may persist until corrected or deleted. A workflow checkpointer is not automatically a semantic-memory system.

Store structured, provenance-bearing state where possible and render it into a prompt when needed. Do not store an opaque prompt string as the only record of what the agent knew or decided.

## 4. Memory record design

For each stored item, define:

- content and memory type;
- source or evidence span;
- owner, tenant, and access-control boundary;
- created time, last-confirmed time, and expiration rule;
- confidence or verification status;
- sensitivity class and deletion path;
- whether the item was inferred by a model or explicitly stated.

For example, “user prefers concise summaries” should not silently become a permanent fact if it was inferred from one short message. Allow correction and deletion.

## 5. Read and write policies

### Writing memory

Before storing:

1. Is this useful beyond the current task?
2. Is it directly stated or inferred?
3. Is it personal, sensitive, or regulated?
4. Can it be stored with a shorter or less identifying form?
5. Should the user approve or be able to review it?

Use extraction thresholds and provenance. A memory write should be idempotent or deduplicated when the same fact is encountered repeatedly.

### Reading memory

Before injecting a memory:

1. Is the current user authorized to see it?
2. Is it relevant to this task?
3. Is it fresh and consistent with newer evidence?
4. Could it conflict with the current request?
5. Can the task proceed without it?

Retrieved memory is evidence with a source and age, not a higher-priority instruction. Do not let an old stored preference override an explicit current request.

## 6. Context budgeting and condensation

Reserve space for the current task, tool schemas, and final response before adding history. Prioritize information in this order, adapted to the task:

1. non-negotiable policy and authorization;
2. current user goal and constraints;
3. directly relevant evidence with provenance;
4. unresolved decisions and current workflow state;
5. the minimum recent history needed for continuity;
6. lower-priority background.

When condensing a long interaction, preserve:

- objective and constraints;
- decisions already made;
- source references and their dates;
- unresolved questions and risks;
- actions completed and actions still pending.

Do not summarize away negations, approval status, quantitative limits, or the difference between an observed fact and an inference. Test summaries by resuming tasks from them and checking whether the next action remains correct.

## 7. Prompt caching as a context optimization

Some model providers can reuse repeated prompt prefixes. A general design principle is to put stable instructions and examples before variable user-specific content. Cache behavior, pricing, and invalidation are provider-specific; verify current provider documentation before relying on it.

Never share a cached prefix across users if it contains user-specific or tenant-specific information. Caching improves repeated computation behavior; it does not reduce the need for authorization and retention controls.

## 8. Persistence, interrupts, and human review

For a workflow that must pause, persist:

- a run or thread identifier;
- minimal state needed to resume;
- the exact pending action and approval question;
- relevant source and tool versions;
- whether the action has already executed.

Resume only after rechecking authorization and whether the approval is still valid. Use idempotency keys or a durable action record so a retry cannot send a duplicate payment, message, or command.

Human review is especially useful immediately before an irreversible or high-impact side effect. Show the reviewer the proposed action, supporting evidence, and consequences. Record approval, edits, rejection, reviewer identity or role, and time.

Time travel or state editing can help debugging, but editing production state can bypass business rules. Restrict it, audit it, and revalidate invariants after an edit.

## 9. Evaluate memory as a product feature

Create paired cases where memory helps, is irrelevant, is wrong, is stale, or belongs to another user. Measure:

- task success with and without memory;
- memory retrieval precision and useful recall;
- contradiction and staleness errors;
- sensitive-data retention and cross-user leakage;
- token, latency, and storage cost;
- success of correction and deletion requests.

A memory that increases response personalization but causes privacy leakage is not a successful memory system.

## Practice with answer checks

1. Is a LangGraph-style checkpoint the same as long-term semantic memory?
2. A user once asks for a one-sentence answer. Should that become a permanent preference?
3. A memory conflicts with the current request. Which should guide the response?
4. What must be rechecked before resuming a paused workflow?
5. Why should a memory include when and where it came from?

### Answers

1. No. A checkpoint restores workflow state; semantic memory stores reusable facts. They have different purposes and retention policies.
2. Not automatically. Treat it as a local task constraint unless repeated, explicitly saved, or confirmed.
3. The explicit current request normally governs, subject to higher-priority safety and authorization policy.
4. Authorization, approval freshness, pending action status, deadlines, and idempotency of side effects.
5. Provenance and age help detect inference, conflict, staleness, and scope violations.

## Mini-project

Design a memory layer for a research assistant. Define two memory types, their schemas, read/write policies, tenant boundary, expiration, correction/deletion path, and a 20-case evaluation set containing stale and cross-user cases. Demonstrate a paused workflow that resumes with an approval check and cannot repeat a side effect after retry.

## Common mistakes

- Treating the whole conversation transcript as useful memory.
- Putting retrieved memory above current instructions or authorization.
- Storing inferred personal facts without a review or deletion path.
- Resuming a workflow without checking whether the pending action already ran.
- Compressing context without testing the decisions made after resumption.

## References

- [Thinking in LangGraph](https://docs.langchain.com/oss/javascript/langgraph/thinking-in-langgraph) — state design, checkpoints, interruptions, retries, and resumption.
- [LangChain learning resources](https://docs.langchain.com/oss/python/learn) — agent memory and workflow tutorials.
