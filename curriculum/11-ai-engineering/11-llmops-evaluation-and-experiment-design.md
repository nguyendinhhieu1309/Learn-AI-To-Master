# LLMOps: Evaluation and Experiment Design

LLMOps applies software and measurement discipline to systems built with language models. The goal is to make changes explainable: what improved, for whom, at what cost, and with which new failure modes?

## Learning objectives

- Build a representative offline evaluation set and protect it from tuning leakage.
- Measure retrieval, answer quality, tool behavior, safety, latency, and cost separately.
- Design controlled comparisons and interpret uncertainty.
- Use model judges as fallible measuring instruments, not ground truth.
- Instrument production systems while limiting sensitive data collection.
- Turn evaluation into a release gate and a continuous improvement loop.

## Prerequisites

Complete [RAG evaluation and observability](../08-nlp-llms/09-rag-evaluation-and-observability.md), the [RAG worked lab](../08-nlp-llms/10-rag-system-worked-lab.md), and [Agent evaluation harness lab](../10-agentic-ai/08-agent-evaluation-harness-lab.md).

## 1. Start with a task contract

Write down the system's job before selecting a metric:

- input types, languages, users, and operating context;
- what counts as a correct, complete, grounded response;
- when the system must ask a question, abstain, or hand off;
- which actions are allowed and which require human approval;
- latency, availability, privacy, and cost limits;
- harm severity and acceptable failure rates by slice.

Separate hard constraints from preferences. For example, a lower average latency cannot compensate for an unauthorized tool call. A weighted score is useful for ranking candidates only after safety and contract constraints pass.

## 2. Build an evaluation dataset that represents use

Create examples from real task patterns, expert-authored cases, and known failures. Cover common traffic and rare high-impact cases. Store a stable ID, input, expected properties or evidence, source/version, slices, and adjudication notes.

Useful slices include:

- language, domain, user type, and request length;
- answerable, ambiguous, and unanswerable questions;
- fresh versus stale evidence;
- permission boundary and risk level;
- tool success, malformed output, timeout, and retry;
- single-step versus multi-step workflows.

Keep development examples for iteration, a validation set for choosing configurations, and a locked test set for final estimates. Do not repeatedly tune prompts against the locked set: that turns it into training data. Version the corpus, parser, index, prompt, model, and test examples so a result can be reproduced.

For small datasets, report counts alongside rates. If 2 of 20 cases fail, that is 10% observed failure, but uncertainty remains large. Inspect exact examples and avoid claims of precise population performance.

## 3. Measure each stage separately

### Retrieval

Use relevance labels to measure Recall@k, Precision@k, reciprocal rank, or nDCG. Also test access filtering, freshness, no-hit behavior, and the percentage of answers whose supporting evidence appears in the retrieved set.

### Answer quality

Score factual correctness, groundedness, completeness, citation support, calibrated abstention, and format validity as distinct fields. A fluent answer can be wrong; a correct answer can still be ungrounded or violate the output contract.

### Agent behavior

Measure task completion, tool selection, argument validity, unnecessary calls, recovery after tool failure, policy compliance, approval handling, and correct stopping. Score trajectories as well as final messages.

### System behavior

Track stage latency, end-to-end p50/p95/p99, timeout rate, tokens, retries, throughput, memory, and cost per successful task. Define success before dividing cost: a cheap response that fails is not a successful unit of work.

## 4. Use model judges carefully

A model judge can help triage large evaluation sets, but its score is an estimate with biases. It may favor longer answers, confident tone, familiar wording, or its own model family. Position order can affect pairwise judgments.

Improve judge reliability:

1. Use a short rubric with observable criteria and examples.
2. Hide model identity and randomize answer order for pairwise comparison.
3. Require claim-level evidence checks for groundedness.
4. Compare a sample of judge decisions with expert human labels.
5. Report disagreement and judge version, prompt, and model.
6. Use deterministic checks for schemas, citations, permissions, and exact constraints.

Do not use a single judge score as a release decision for safety-critical behavior. Escalate high-impact slices for qualified human review.

## 5. Design controlled experiments

Compare systems on the same examples and conditions. Change one major factor where possible: chunking, retriever, reranker, prompt, model, context budget, or tool policy. If several factors change together, use a deliberate factorial design or do not claim which change caused the result.

For paired outcomes, define the per-example difference. For a quality score:

$$
d_i = s_{i,\mathrm{candidate}} - s_{i,\mathrm{baseline}}.
$$

Report the mean or median paired difference, a confidence interval or bootstrap interval, the number of examples, and slice-level changes. For binary outcomes, show counts of candidate wins, losses, ties, and failures. Small gains with wide uncertainty need more evidence.

Example experiment matrix:

| Variant | Retrieval | Reranker | Main question |
| --- | --- | --- | --- |
| A | Dense | None | Baseline |
| B | BM25 + dense with rank fusion | None | Does hybrid search improve evidence recall? |
| C | BM25 + dense with rank fusion | Cross-encoder | Does reranking improve answer quality enough to justify latency? |

Hold the corpus snapshot, generation model, prompt, and test questions constant. Record index size, retrieval latency, end-to-end latency, token count, and cost beside quality.

## 6. Release gates and rollout

Use a gate in stages:

1. **Contract checks:** schemas, access controls, citation IDs, prohibited actions.
2. **Regression set:** previously observed failures must not silently return.
3. **Quality and safety slices:** critical slices meet their own thresholds.
4. **Performance and cost:** service budgets stay within limits.
5. **Human review:** high-impact or newly changed behaviors are inspected.
6. **Canary:** expose the candidate to limited traffic with rollback criteria.

Choose thresholds from product risk and operational requirements; there is no universal acceptable accuracy number. If the regression set is tiny, treat it as a smoke alarm, not a statistical guarantee.

## 7. Production traces and privacy-aware observability

Record enough information to diagnose behavior: request and trace IDs, prompt/template version, model version, retrieval IDs and ranks, tool names and validated arguments, stage timings, token counts, validation outcomes, retries, and final status.

Raw prompts and retrieved documents can contain personal, confidential, or regulated data. Prefer metadata and redacted samples; restrict access, retention, and export; avoid logging secrets; and sample only when full content is needed and approved by the data policy. A trace system must have its own access control and retention plan.

Monitor both system health and task-quality indicators:

- error, timeout, and fallback rates;
- retrieval no-hit and citation-validation rates;
- abstention, escalation, and tool-denial rates;
- latency and cost distributions;
- user feedback and delayed human-reviewed samples;
- drift in request mix and performance by slice.

Feedback such as clicks, thumbs-up, or task completion is a proxy, not necessarily a correctness label. Check for selection bias: users who respond may differ from those who do not.

## 8. Failure analysis and learning loop

For each failed case, record the earliest failure stage, impact, reproducibility, affected slice, and a proposed intervention. Common categories include missing or stale source data, parsing, retrieval, context packing, unsupported generation, tool contract failure, permission failure, timeout, and evaluation defect.

Then:

1. preserve the failing example and minimal trace;
2. reproduce it against the saved system versions;
3. identify whether the failure is a bug, data issue, model behavior, or metric gap;
4. make one targeted change;
5. rerun regression, safety, slice, latency, and cost checks;
6. release gradually and watch predefined rollback signals.

Never remove a difficult example just because the new version performs poorly. Correct a mislabeled or invalid case with a documented adjudication.

## Practice with answer checks

1. Hybrid retrieval improves Recall@10 but adds 180 ms. What else must be measured before adoption?
2. A candidate beats baseline on a model judge but loses on human review of citations. What should happen?
3. Why is a production thumbs-up not equivalent to an expert correctness label?
4. What does repeatedly tuning on a locked test set do?
5. A change improves average quality but increases unauthorized tool attempts. How should the release decision be framed?

### Answers

1. Answer grounding and correctness, latency distribution (not only average), cost, index and retrieval load, and relevant query slices.
2. Investigate the disagreement, inspect claim-to-source support, calibrate the judge, and do not let the judge override a critical grounding failure.
3. Feedback is self-selected and may reflect speed, tone, or satisfaction rather than factual accuracy.
4. It leaks information from the test set into design decisions, making the final estimate optimistic.
5. Treat authorization as a hard constraint. The candidate fails the contract until the unsafe behavior is fixed and retested.

## Mini-project

Choose a small RAG or tool-using application. Build a versioned evaluation set with at least four slices, including unanswerable or denied requests. Compare a baseline with one change using paired examples. Report stage-specific quality, failures, uncertainty, p95 latency, cost per successful task, and a release recommendation. Include one privacy-safe trace schema and explicit release/rollback criteria.

## Common mistakes

- Reporting only one aggregate quality score.
- Comparing variants on different questions or corpus snapshots.
- Treating a model judge as ground truth.
- Reporting percentage changes without example counts or uncertainty.
- Logging raw prompts indefinitely by default.
- Releasing on average quality while a critical safety slice regresses.

## Further study

- [RAGAS metrics](https://docs.ragas.io/en/v0.4.3/concepts/metrics/available_metrics/)
- [BEIR retrieval benchmark](https://arxiv.org/abs/2104.08663)
- [OpenTelemetry documentation](https://opentelemetry.io/docs/)
