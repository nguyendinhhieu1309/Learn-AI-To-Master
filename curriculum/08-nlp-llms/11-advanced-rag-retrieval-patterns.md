# Advanced RAG Retrieval Patterns

Advanced retrieval is a response to a diagnosed failure. Do not add a graph database, query rewriting, or a reranker before measuring which questions the baseline system misses and why.

## Learning objectives

- Match chunking and indexing strategies to document structure.
- Combine lexical and semantic search using rank fusion.
- Explain query rewriting, decomposition, reranking, and diversity selection.
- Choose when graph or multimodal retrieval is justified.
- Design ablations that attribute quality, latency, and cost changes.

## Prerequisites

Complete [retrieval engineering](05-retrieval-engineering.md), the [RAG system lab](10-rag-system-worked-lab.md), and [RAG evaluation](09-rag-evaluation-and-observability.md).

## 1. Diagnose retrieval failure first

Label failed questions by cause:

- relevant document is absent from the corpus;
- parser omitted text, table cells, captions, or layout;
- chunk split a definition from its condition or heading;
- query uses different terminology from the document;
- relevant passage was retrieved but ranked too low;
- relevant passage was retrieved but lost during context packing;
- evidence is conflicting, stale, or unauthorized.

Each cause suggests a different intervention. Increasing the number of retrieved chunks will not fix a document that was never indexed.

## 2. Chunking and indexing strategies

### Fixed or recursive chunks

Use for a simple baseline and relatively uniform prose. Keep paragraph and heading boundaries when possible. Measure chunk size in the representation that matters to the model, not only characters.

### Semantic chunking

Use changes in topic or discourse structure to decide boundaries. It can preserve coherent ideas but adds a model or heuristic and may create variable-size chunks. Compare against a simpler splitter on labeled questions.

### Parent-child retrieval

Index small child chunks for precise matching, then return the larger parent section for context. Store both IDs and provenance so a child can be expanded without losing its source location.

### Summary or hierarchical indexes

Index summaries at document or section level to route broad questions, then retrieve detailed passages for evidence. A summary is a navigation aid, not a replacement for source evidence.

### Late chunking

For compatible embedding systems, encode a larger contextual span before pooling token representations into smaller chunk vectors. This can preserve neighboring context, but depends on model support, memory, and a careful mapping from vectors back to source spans.

## 3. Approximate nearest-neighbor tradeoffs

Large vector collections often use approximate nearest-neighbor (ANN) indexes. A hierarchical navigable small-world (HNSW) index builds a graph that searches promising neighborhoods without comparing every vector.

Common HNSW controls express tradeoffs:

- **M:** graph connectivity; higher values generally increase memory and can improve navigability.
- **ef_construction:** candidate breadth while building the graph; higher values cost more build time and may improve index quality.
- **ef_search:** candidate breadth during queries; higher values usually improve recall at additional query cost.

Parameter names and defaults vary by vector store. Measure recall against exact search on a representative sample before tuning the index. Report memory, build time, query latency, and recall together.

## 4. Hybrid lexical and dense retrieval

Dense embeddings can match paraphrases but may miss exact identifiers, rare names, error codes, and unusual strings. Lexical retrieval such as BM25 can be strong on exact terms but may miss semantic paraphrases.

A robust baseline runs both and merges ranked lists. Reciprocal Rank Fusion (RRF) assigns:

$$
\operatorname{RRF}(d)=
\sum_{r\in\mathcal R}\frac{1}{k+\operatorname{rank}_r(d)},
$$

where $\mathcal R$ is the set of ranked result lists and $k$ dampens the effect of very high ranks. RRF does not require raw score calibration across retrievers.

Evaluate lexical, dense, and hybrid variants on the same query set. Preserve the score and rank from each channel for debugging.

## 5. Query transformation

### Multi-query generation

Create a few alternative search phrasings and merge their results. This can improve recall for vocabulary mismatch, but it can add irrelevant queries and duplicate documents.

### Hypothetical Document Embeddings (HyDE)

Ask a model to draft a hypothetical passage that might answer the query, then use its embedding as a retrieval query. The hypothetical passage is a search aid, **not evidence**. Hallucinated details in it must never be shown as retrieved facts.

### Query decomposition

Break a multi-part question into answerable subquestions, retrieve evidence for each, and check that every required part is covered before synthesis. Keep the original query so decomposition does not silently change user intent.

### Step-back and routing

A broader step-back query can retrieve background concepts; a router can choose a collection or retriever based on task type. Add these only when separate evaluation shows a benefit and a safe fallback exists.

## 6. Post-retrieval processing

### Cross-encoder reranking

A bi-encoder embeds query and document separately, which is efficient for large candidate sets. A cross-encoder scores a query-document pair jointly, which can rank a smaller candidate set more precisely but costs more per pair.

Use a two-stage pipeline:

1. retrieve a broad candidate set cheaply;
2. rerank a bounded number of candidates;
3. return only the most useful evidence under the context budget.

Evaluate gains against added latency and cost.

### Maximal marginal relevance (MMR)

MMR balances relevance to the query with diversity among selected passages:

$$
\operatorname{MMR}(d)=
\lambda\,\operatorname{sim}(q,d)
-(1-\lambda)\max_{s\in S}\operatorname{sim}(d,s),
$$

where $S$ is the already-selected set. Larger $\lambda$ favors direct relevance; smaller $\lambda$ penalizes redundancy more strongly. Diverse context helps only when it preserves necessary evidence.

### Contextual compression and ordering

Compression can remove irrelevant sentences from retrieved passages, but it can also remove a negation, date, exception, or citation anchor. Keep the original passage and compare compressed evidence with it.

Test context ordering. Some models use information near the beginning or end of a long context more reliably than information buried in the middle. This is model- and prompt-dependent; do not treat one ordering rule as universal.

## 7. Graph retrieval

Graph-based retrieval can help when questions depend on entities and relations across documents: ownership, dependency chains, timelines, or multi-hop connections.

A graph RAG pipeline commonly:

1. extracts entities and relations from source documents;
2. validates entity identity and provenance;
3. stores a graph with links back to source spans;
4. maps a question to a bounded graph query or traversal;
5. retrieves source evidence for the resulting entities and relations;
6. generates a response with citations.

Entity extraction can be wrong, entity resolution can merge distinct people, and graph completeness is difficult to guarantee. Every edge used in an answer should be traceable to source evidence. Graph retrieval is unnecessary for many direct fact lookup tasks.

## 8. Multimodal retrieval

Text-only indexes lose information from diagrams, scanned forms, charts, and images. Possible approaches include:

- OCR and layout extraction;
- image captions indexed as text;
- shared image-text embeddings;
- visual-language retrieval over page images or crops.

Choose the simplest approach that preserves the evidence needed by the task. OCR can lose spatial relationships; captions can omit small labels; image embeddings can be harder to interpret and cite. Preserve page, crop, and bounding-box provenance so a reviewer can inspect the visual evidence.

## 9. A controlled optimization sequence

Change one layer at a time:

1. repair parsing and metadata;
2. compare chunk boundaries;
3. compare lexical, dense, and hybrid retrieval;
4. tune candidate count and ANN recall;
5. try query transformation;
6. rerank and deduplicate;
7. adjust context packing and ordering;
8. try graph or multimodal methods only for appropriate query categories.

For each change, report retrieval recall/precision, answer grounding and relevance, latency, token use, cost, and failure categories. Keep a no-answer and authorization slice in the evaluation set.

## Practice with answer checks

1. Exact product IDs are missed by vector search. Which retrieval channel should you compare?
2. A HyDE passage contains a plausible but unsupported date. Can the answer cite it?
3. When is parent-child retrieval useful?
4. What is the main tradeoff of cross-encoder reranking?
5. Which kind of question is a strong candidate for graph retrieval?
6. Why preserve source spans after contextual compression?

### Answers

1. Compare lexical/BM25 retrieval and a hybrid merge with dense retrieval.
2. No. The generated passage is a query aid, not source evidence.
3. When small chunks improve matching but a larger section is needed to answer with sufficient context.
4. Better joint query-document ranking over a smaller candidate set, at greater compute and latency per pair.
5. A question requiring relations across several entities or documents, such as finding a dependency path or tracing ownership over time.
6. A reviewer needs to verify the answer against the original; compression may omit qualifiers or citation anchors.

## Mini-project

For one corpus, compare a recursive-chunking dense baseline, hybrid retrieval with RRF, and one post-retrieval method. Add semantic or parent-child chunking only if a labeled failure category motivates it. Report a fixed query set, retrieval and answer metrics separately, p95 latency, cost, and at least ten retrieved-context failures.

## Common mistakes

- Adding advanced retrieval before measuring baseline failures.
- Treating a generated hypothetical answer as evidence.
- Comparing raw scores from unrelated retrievers without calibration.
- Reporting answer quality without retrieval recall.
- Using GraphRAG for simple fact lookup without demonstrating a need.
- Ignoring index-build cost, memory, access filtering, and citation provenance.
