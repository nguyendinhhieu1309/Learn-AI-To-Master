# Retrieval Engineering

## Learning objectives

Build a retrieval pipeline that returns useful evidence, diagnose retrieval failures separately from generation failures, and maintain traceable sources.

## 1. Indexing pipeline

1. Confirm source permissions and capture provenance/version.
2. Extract text and metadata (title, author, date, section, page, ACL).
3. Normalize carefully; preserve headings, tables, code, and language cues.
4. Split documents into passages that preserve meaning and metadata.
5. Embed passages or build a lexical index; record embedding model/version.
6. Store the index and a reproducible manifest so it can be rebuilt.

## 2. Chunking is a trade-off

Small chunks are focused but may omit definitions or surrounding conditions. Large chunks preserve context but can bury the relevant sentence and consume the context window. Split at semantic boundaries where possible; overlap only when it prevents boundary loss. Compare chunking settings on a reviewed query set rather than choosing an arbitrary token count.

## 3. Retrieval methods

- **Lexical search** (BM25/inverted index): strong for exact phrases, identifiers, and rare terms.
- **Dense/vector search:** maps queries and passages to embeddings and ranks by similarity; can match paraphrases but may miss exact identifiers or negation.
- **Hybrid retrieval:** combines lexical and semantic candidates.
- **Reranking:** a stronger model scores a small candidate set for relevance; adds quality and cost/latency.

Apply metadata filters and access control before returning passages. A post-generation filter is too late if unauthorized text was already placed in the model context.

## 4. Evaluate retrieval independently

Create questions with known supporting passages. Measure recall@k (whether relevant evidence appears in the top k), precision/ranking metrics, and performance by document type/language. Inspect cases with no relevant passage. Retrieval failure may come from extraction, chunking, query formulation, embeddings, filters, or stale indexing.

## 5. Generate grounded responses

Pass only relevant evidence with source IDs and instruct the model to answer from the evidence. Require citations that resolve to stored source spans. Verify citations programmatically and sample-check whether each cited span supports its claim. If evidence is absent, contradictory, or stale, ask a question, qualify the response, or abstain.

## 6. Maintain the index

Handle document updates and deletions, duplicate sources, permissions, stale embeddings, and version migrations. Log index and embedding-model versions with each run. Respect retention and delete derived vectors when the underlying source must be removed.

## Practice

Build a 20-question evaluation set over a small document collection. Include exact lookup, paraphrase, multi-document, unanswerable, and permission-restricted questions. Compare lexical, vector, and hybrid retrieval. Label the failure stage for each error.

## Common mistakes

- Tuning generation prompts when the required passage was never retrieved.
- Ignoring source ACLs during indexing or query filtering.
- Treating a high vector similarity as proof that two passages mean the same thing.
- Keeping stale or deleted text in embeddings and caches.
