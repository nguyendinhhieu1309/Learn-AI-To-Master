# RAG Evaluation and Observability

Retrieval-augmented generation (RAG) combines search with a language model. Evaluation must locate whether a failure came from retrieval, evidence selection, generation, or the user’s question.

## 1. Define the task before choosing metrics

Create a representative test set containing questions, expected answer properties, source documents, and acceptable abstention cases. Include unanswerable questions, ambiguous questions, multi-hop questions, long documents, outdated sources, and adversarial wording. Keep a locked test set separate from prompt and chunking experiments.

Decide what a correct answer means: factual correctness, groundedness in supplied evidence, completeness, citation validity, and appropriate refusal may be separate dimensions. A concise answer can be correct even when it differs lexically from a reference answer.

## 2. Evaluate retrieval independently

For each query, label relevant evidence passages or documents. Useful measures include:

- Recall@k: whether relevant evidence appears in the top $k$ results.
- Precision@k: fraction of retrieved top-$k$ items judged relevant.
- Mean reciprocal rank: how high the first relevant item appears.
- nDCG: ranking quality when relevance has graded labels.

Test exact, semantic, and metadata-sensitive queries. Measure retrieval latency and index freshness. A high answer score on a small set does not prove retrieval quality if the generation model already memorized the answer.

## 3. Evaluate generated answers

Score answer correctness, evidence support, completeness, citation precision, citation recall, and abstention behavior. Citation precision asks whether cited passages support the attached claims; citation recall asks whether material claims have adequate citations. A citation that merely exists in the retrieved context is not necessarily valid evidence.

Use deterministic checks when possible: schema validity, required fields, citation IDs present in context, and forbidden content. Use human review for nuanced judgments and calibrate any model-based judge against human labels. Blind reviewers to model identity when comparing systems.

## 4. Failure attribution

Classify each failure at the earliest broken stage:

1. Query interpretation or rewriting changed the intent.
2. Search failed to retrieve relevant evidence.
3. Reranking omitted or misordered useful passages.
4. Chunking separated a fact from its context.
5. The prompt did not require evidence or abstention.
6. The model ignored, misunderstood, or contradicted evidence.
7. Citation parsing or rendering attached evidence incorrectly.

This classification directs work. Increasing model size will not fix a missing document, and improving recall can worsen precision or increase context noise.

## 5. Ablations and change control

Change one major component at a time when diagnosing a pipeline: embedding model, chunk size, overlap, metadata filter, reranker, context budget, prompt, or generation model. Keep a baseline and record dataset/index snapshots. Compare quality, latency, and cost together. Run slice metrics for language, document age, source, and question type.

## 6. Online observability

Record safe, privacy-aware traces of query transformation, retrieved document IDs and scores, prompt/version, model/version, token counts, latency by stage, tool calls, output validation, and user feedback. Avoid retaining sensitive raw content when it is not needed; redact or control access where content must be retained for debugging.

Monitor retrieval no-hit rates, citation validation failures, abstention rates, latency, cost, and feedback trends. Delayed human labels are needed to estimate real answer quality; user clicks and thumbs-up are biased proxies.

## 7. A practical evaluation loop

1. Build an error taxonomy and baseline test set.
2. Measure retrieval and answer quality separately.
3. Review representative failures, including apparent successes with unsupported claims.
4. Make a targeted change and rerun the locked regression set.
5. Test new examples for uncovered failure categories.
6. Canary deploy and monitor quality proxies, service health, cost, and feedback.
7. Sample production traces for privacy-reviewed human evaluation.

## 8. Exercises

1. The answer is wrong and the correct paragraph never appears in top-20 results. Which subsystem should be investigated first?
2. Retrieval recall improves while answer correctness drops. Give two plausible explanations.
3. A citation points to a retrieved document but the passage contradicts the claim. Which citation metric catches the problem?
4. Why can an LLM judge give misleading confidence when evaluating answers from the same family of models?

## References

- [RAGAS paper](https://arxiv.org/abs/2309.15217)
- [BEIR benchmark](https://arxiv.org/abs/2104.08663)
- [OpenTelemetry documentation](https://opentelemetry.io/docs/)
