# NLP, LLMs, and Generative AI

## Outcomes

- Represent text and understand tokenization, embeddings, and classical NLP baselines.
- Explain transformers, language-model training, inference, and adaptation.
- Build and evaluate retrieval-augmented and generative applications.
- Apply safety, cost, and evaluation practices across the LLM lifecycle.

## Prerequisites

Python, data processing, and basic ML. Deep learning and probability help with transformer training and fine-tuning.

## Suggested sequence

1. [Text representations and transformers](01-text-representations-and-transformers.md)
2. [Classical NLP, sequence models, and evaluation](02-classical-nlp-sequence-models-and-evaluation.md)
3. [LLM foundations and inference](03-llm-foundations-and-inference.md)
4. [Prompt design and structured outputs](04-prompt-design-and-structured-outputs.md)
5. [Retrieval engineering](05-retrieval-engineering.md)
6. [Fine-tuning and model adaptation](06-fine-tuning-and-model-adaptation.md)
7. [LLM evaluation, safety, and cost](07-llm-evaluation-safety-and-cost.md)
8. [RAG, evaluation, and safety](08-rag-evaluation-and-safety.md)
9. [RAG evaluation and observability](09-rag-evaluation-and-observability.md)
10. [RAG system worked lab](10-rag-system-worked-lab.md)
11. [Advanced RAG retrieval patterns](11-advanced-rag-retrieval-patterns.md)

## Practice

- Build a TF-IDF baseline for a text classification task.
- Compare tokenization on code, multilingual text, and rare words.
- Build a small search index and inspect retrieved passages before generating a response.
- Write test cases for unsupported questions, ambiguity, and prompt injection attempts.
- Compare a TF-IDF text classifier with a transformer baseline using the same split.
- Change one retrieval setting at a time and measure both answer quality and retrieval recall.
- Complete the [RAG system lab](10-rag-system-worked-lab.md), including permissions, no-answer queries, and citation audits.
- Complete the [advanced RAG comparison](11-advanced-rag-retrieval-patterns.md) with a fixed query set before adopting a more complex retrieval design.

## Checkpoint project

Create a question-answering tool over a small, well-sourced document collection. Measure retrieval separately from answer quality, include citations to source passages, and document where the system should decline to answer.

Also compare prompting, retrieval, and fine-tuning as solutions to different task requirements; record quality, latency, and cost for each approach.
