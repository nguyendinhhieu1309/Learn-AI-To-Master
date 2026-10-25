# LangChain and LangGraph

## Learning objectives

Choose between a higher-level agent framework and an explicit graph runtime, build a small tool-calling agent, and understand checkpointed human approval.

> **Version note:** Framework APIs move quickly. The linked official documentation and examples were reviewed on 2026-10-26; check your installed version before copying a code sample into a project.

## 1. How the pieces fit

LangChain provides model/tool integrations and higher-level agent patterns. Its current `create_agent` API packages a model-and-tool loop and can be extended with middleware. Current LangChain documentation describes that agent loop as running on LangGraph under the hood.

LangGraph is a lower-level orchestration framework/runtime for explicit stateful workflows. A graph has **state**, **nodes** that read/update state, and **edges** that control what runs next. It supports combining deterministic nodes with model-driven nodes, loops, persistence, streaming, and human-in-the-loop pauses. LangGraph can be used without LangChain.

LangSmith is an optional product for tracing/evaluation and related lifecycle work in this ecosystem. You can also connect other tracing and evaluation systems.

## 2. A small LangChain tool agent

This example follows the current Python API shape; install a provider integration and pin compatible versions for a real project. The model name is a placeholder because provider identifiers change.

```python
from langchain.agents import create_agent
from langchain.tools import tool

@tool
def lookup_order(order_id: str) -> str:
    """Return a read-only status for one order ID."""
    # Replace with an authorized database lookup; never trust the ID alone.
    return f"Order {order_id}: status unavailable in this example"

agent = create_agent(
    model="provider:model",
    tools=[lookup_order],
    system_prompt="Use the read-only lookup when the user asks about an order."
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "Check order A-1041."}]
})
```

Type hints and the docstring communicate the tool schema and purpose to the framework/model. The application still must validate identity, tenant ownership, timeouts, and returned data. A type-valid tool call may still be unauthorized or semantically wrong.

Middleware can inspect/change execution around model and tool steps. Common uses include logging, prompt/context changes, retries/fallbacks, rate limits, PII checks, summarization, and human approval. Verify the exact ordering and which paths a middleware applies to in your framework version.

## 3. When to use LangGraph directly

Use an explicit graph when the workflow has meaningful deterministic stages, multiple routes, loops that need bounds, parallel branches, approval/resume points, or durable state. Keep simple request-response tasks simple; graph code is useful when it makes behavior clearer and more controllable.

```python
from typing_extensions import TypedDict
from langgraph.graph import StateGraph, START, END

class State(TypedDict):
    question: str
    evidence: str
    answer: str

def retrieve(state: State):
    return {"evidence": "Evidence retrieved for: " + state["question"]}

def answer(state: State):
    return {"answer": f"Answer from evidence: {state['evidence']}"}

builder = StateGraph(State)
builder.add_node("retrieve", retrieve)
builder.add_node("answer", answer)
builder.add_edge(START, "retrieve")
builder.add_edge("retrieve", "answer")
builder.add_edge("answer", END)
graph = builder.compile()
```

This graph uses placeholder retrieval/answer nodes to make control flow visible. Replace them with real components and define validation/error paths; do not interpret the placeholder text as evidence.

## 4. Persistence and human-in-the-loop

LangGraph checkpointers persist graph state for a thread; a store can retain application data across threads. An in-memory saver is useful in a lesson but does not survive process restart. Production needs an appropriate persistent backend, retention policy, access controls, and state-size management.

An interrupt can pause execution and request external input. Resuming uses the same thread ID and a resume value. A node may restart from its beginning after resumption, so code before the interrupt should be safe to rerun; put side effects behind an approval/execution boundary and use idempotency.

## 5. Framework choice

| Need | Consider |
| --- | --- |
| Common model + tools loop and integrations | LangChain agent abstraction |
| Explicit state machine, branching, persistence, pause/resume | LangGraph |
| Small opinionated Python primitives for agents, tools, handoffs, guardrails, and tracing | OpenAI Agents SDK |
| Standardized connection between an AI host and external tools/resources | MCP |
| Task-specific reusable instructions and optional assets/scripts | Agent Skills |

These categories overlap but are not interchangeable. Start with the smallest layer that matches the control you need. Keep core task logic and evaluation portable when possible.

## Practice

Implement a “search → draft answer → validate citations → human approval → publish” workflow. Mark which steps must be deterministic, which can use an LLM, which need persistent state, and which must pause. Draw the graph before coding it.

## Common mistakes

- Assuming LangChain and LangGraph are competing synonyms; one provides higher-level components, the other explicit orchestration/runtime.
- Using an in-memory checkpointer as durable production memory.
- Treating middleware or a framework's human-approval feature as a complete security design.
- Copying a code example without pinning the library/provider versions or checking current docs.

## Further reading

- [LangChain agents](https://docs.langchain.com/oss/python/langchain/agents)
- [LangChain tools](https://docs.langchain.com/oss/python/langchain/tools)
- [LangChain middleware](https://docs.langchain.com/oss/python/langchain/middleware)
- [LangGraph overview](https://docs.langchain.com/oss/python/langgraph/overview)
- [LangGraph persistence](https://docs.langchain.com/oss/python/langgraph/persistence)
- [LangGraph interrupts](https://docs.langchain.com/oss/python/langgraph/interrupts)
- [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/)
