# RAG System Worked Lab: Retrieval, Grounding, and Evaluation

Retrieval-augmented generation (RAG) connects a language model to a changing or private knowledge source. The system is only as reliable as its document pipeline, retrieval boundary, evidence handling, and evaluation.

## Learning objectives

- Define a grounded-answer contract with explicit abstention behavior.
- Build a document ingestion and retrieval plan that preserves provenance.
- Separate retrieval quality from answer quality.
- Evaluate citations, access control, prompt-injection resistance, latency, and cost.

## Prerequisites

Complete [retrieval engineering](05-retrieval-engineering.md), [prompt design](04-prompt-design-and-structured-outputs.md), and [LLM evaluation](07-llm-evaluation-safety-and-cost.md). The [agent harness lab](../10-agentic-ai/08-agent-evaluation-harness-lab.md) adds execution and trace controls for tool-using systems.

## 1. Write the answer contract

Example contract:

> Answer questions about the current engineering handbook using only passages the user is authorized to read. Cite the passages that support factual claims. If the evidence is missing, stale, or contradictory, say so and identify what needs clarification.

Specify:

- which source collections are in scope;
- how document versions and access permissions are determined;
- what counts as an adequate citation;
- when the system should abstain;
- whether answers may summarize across multiple documents;
- what the user can do if an answer is missing.

## 2. Preserve source identity through ingestion

Each indexed chunk should retain enough metadata to locate and authorize its source:

- stable document ID and version;
- source URI or repository path;
- page, section, or character span;
- owner or access-control label;
- effective date and last-updated time;
- ingestion timestamp and parser version.

Do not create chunks that can no longer be traced back to an authorized source. If permissions change, the retrieval layer must enforce the current policy; hiding a citation after retrieval is not access control.

## 3. Chunk documents as a retrieval experiment

Chunk size and overlap trade off context completeness against retrieval precision and index size.

- Very small chunks can retrieve a definition while omitting its conditions.
- Very large chunks can dilute ranking and consume the context budget.
- Overlap can preserve boundary context but create many near-duplicate results.
- Headers, tables, code, and legal clauses often need structure-aware handling.

Compare at least two chunking strategies on the same labeled questions. Record chunk boundaries and keep document metadata attached. Do not choose a strategy because a few hand-written prompts look good.

## 4. Build retrieval in stages

1. **Query normalization:** preserve user intent; expand abbreviations only when safe.
2. **Authorization filter:** restrict the search space before exposing results.
3. **Candidate retrieval:** compare lexical, dense-vector, or hybrid retrieval.
4. **Candidate ranking:** optionally rerank a small candidate set.
5. **Deduplication and diversity:** avoid filling the context with repeated passages.
6. **Context packing:** include the best evidence within a fixed token budget.
7. **Grounded generation:** require claims to be tied to retrieved evidence.
8. **Citation validation:** confirm each cited source exists in the retrieved set.

These components should be measured separately. A fluent answer cannot recover a fact that retrieval never supplied.

## 5. Evaluate retrieval independently

Create a query set with labeled relevant document IDs or spans. Measure:

$$
\operatorname{Recall@k}
=\frac{\text{queries with at least one relevant result in top }k}
{\text{queries}},
$$

along with a ranking metric such as reciprocal rank or normalized discounted cumulative gain when relevance is graded.

Include:

- exact wording and paraphrases;
- acronym and spelling variants;
- questions that need two sources;
- no-answer queries;
- stale and conflicting documents;
- queries across different permission groups.

Retrieval metrics depend on annotation quality. Review a sample of “misses”; a missing relevant document label can make a good result look bad or the reverse.

## 6. Evaluate answer quality separately

For each answer, assess:

- correctness against the permitted source;
- coverage of required points;
- unsupported or contradicted claims;
- citation existence and citation support;
- appropriate abstention;
- readability and actionability.

Use an explicit rubric with examples of pass and fail. An automated evaluator can help triage results, but its scores are measurements with error, not ground truth. Human review remains important for high-impact use.

## 7. Citation audit

For every factual sentence:

1. Does the citation point to a source that was actually retrieved?
2. Does the cited span support the whole claim or only one clause?
3. Is the source version current for the question?
4. Are two sources in conflict?
5. Is the answer using information outside the authorized corpus?

A syntactically valid citation can still be irrelevant. Track citation support, not only citation presence.

## 8. Security and privacy tests

Treat retrieved text as untrusted data. Add cases where a document says to ignore previous rules, reveal secrets, or invoke a tool. The model should summarize that text as content when relevant, not follow it as an instruction.

Test:

- cross-tenant retrieval and document authorization;
- prompt injection in documents and query parameters;
- hidden or malformed content in PDFs and HTML;
- source deletion and permission revocation;
- personal data in logs and traces;
- citations that point to a document the user cannot access.

Authorization belongs in the retrieval boundary and must be checked again when opening the source.

## 9. Latency, context, and cost

Measure each stage:

$$
T_{\mathrm{request}}=
T_{\mathrm{query}}+
T_{\mathrm{retrieve}}+
T_{\mathrm{rerank}}+
T_{\mathrm{generate}}+
T_{\mathrm{validate}}.
$$

Track p50/p95/p99 latency, retrieved token count, model tokens, retries, and cost per successfully answered task. A lower retrieval $k$ may reduce cost but hurt recall; a larger context may increase latency without improving evidence.

## 10. Evaluation plan

Start with 40–100 labeled questions if the corpus and budget allow. Include answerable, unanswerable, multi-hop, stale, conflicting, and access-control cases. Keep a development set for iteration and a held-out regression set for final review.

For each experiment, record:

- corpus snapshot and parser/chunker versions;
- embedding and ranking configuration;
- prompt and model version;
- retrieval metrics;
- answer rubric scores and failure categories;
- latency and cost;
- known limitations and evaluator version.

## Practice with answer checks

1. Retrieval recall is low, but answers that receive correct context are accurate. Which component should be improved first?
2. Retrieval recall is high, but unsupported claims remain. Which stages should be inspected?
3. Why should permission filtering occur before the model sees retrieved text?
4. What does Recall@5 fail to tell you?
5. Why is citation presence not the same as citation support?

### Answers

1. Improve document parsing, chunking, query handling, candidate retrieval, or ranking before changing generation.
2. Inspect context packing, prompt constraints, source conflicts, answer generation, and citation validation.
3. Once unauthorized text reaches the model or logs, later removal cannot undo exposure.
4. It does not describe ranking within the top five, answer correctness, citation quality, authorization, or latency.
5. A citation may exist but fail to entail the sentence it is attached to.

## Mini-project

Build a read-only RAG prototype over a small, licensed document set. Label at least 40 questions with evidence spans. Compare two chunking or retrieval configurations, report retrieval and answer metrics separately, include no-answer and prompt-injection cases, validate citations, and document p95 latency and cost. Preserve a held-out regression set.

## Common mistakes

- Blaming generation for a retrieval miss.
- Optimizing chunk size on a handful of hand-picked prompts.
- Treating a model-generated citation as verified evidence.
- Ignoring document version and access-control metadata.
- Evaluating only answerable queries.
- Using an LLM judge without calibration or human spot checks.

## References

- [LangChain learning resources](https://docs.langchain.com/oss/python/learn) — RAG and agent tutorials across LangChain and LangGraph.
- [LangChain retrieval concepts](https://docs.langchain.com/oss/python/langchain/retrieval) — retrieval components and how they fit together.
