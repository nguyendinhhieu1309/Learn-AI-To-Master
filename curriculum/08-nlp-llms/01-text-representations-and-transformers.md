# Text Representations and Transformers

## Learning objectives

Explain how text becomes model input, describe the main idea of a transformer, and recognize the limits of language-model outputs.

## 1. Text is data with structure

Start by defining the task: classify a review, find an answer in documents, translate, or generate text. Preserve useful structure such as punctuation, language, code blocks, and document boundaries. Normalization (lowercasing, removing markup, standardizing whitespace) can help some tasks but can also remove meaning, such as capitalization in names or case-sensitive code.

## 2. Tokens and embeddings

A tokenizer splits text into units called **tokens**. Tokens may be whole words, subwords, characters, or bytes. Token IDs are integers, not meanings. An embedding table maps each token ID to a vector the model can process. Tokenization affects context length, rare words, multilingual text, and cost.

Classical methods include bag-of-words and TF-IDF. TF-IDF gives higher weight to terms frequent in one document but less common across documents. A linear model over TF-IDF is a useful, inexpensive baseline for many text classification tasks.

## 3. Attention and transformers

Self-attention lets each token gather information from other tokens based on learned query/key similarity. Transformer blocks combine attention and a feed-forward network, usually with residual connections and normalization. Positional encodings or position embeddings provide order information. A causal language model masks future tokens and predicts the next token; an encoder model may use context from both directions for classification or representation.

Pretraining learns broad statistical patterns from a large text corpus. Fine-tuning adapts a model to a task using examples. Neither guarantees factual knowledge or fairness. Training data, label quality, prompt, and decoding all affect outputs.

## 4. Generation is prediction, not verification

Autoregressive generation repeatedly samples or selects a next token. Temperature and top-k/top-p controls alter output diversity, not truth. A fluent answer can still be false, stale, or unsupported. For factual tasks, provide relevant evidence and verify that output claims follow from it.

## 5. Choose a baseline and measure

For text classification, start with TF-IDF + logistic regression. For search, start with lexical search. Compare a neural model only after defining the metric and representative test set. Track languages, document types, and user populations; aggregate performance can hide uneven support.

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression
from sklearn.pipeline import make_pipeline

texts = ["easy to use", "works very well", "not useful", "poor quality"]
labels = [1, 1, 0, 0]
baseline = make_pipeline(TfidfVectorizer(), LogisticRegression())
baseline.fit(texts, labels)
print(baseline.predict(["useful and easy"]))
```

This tiny example demonstrates a pipeline, not a meaningful evaluation: four examples are far too few for a performance claim. In a real task, use a representative split and report errors by class and language.

## Mini exercise

Take a sentence with a rare name, an emoji, and a code identifier. Compare how two tokenizers split it. Explain why token count and token meaning are different things. Then write one claim a generative model could make from the sentence and what evidence would be needed to verify it.

## Common mistakes

- Treating token IDs as words or their numeric values as semantic distance.
- Removing punctuation or case without checking whether the task depends on it.
- Assuming a model that produces fluent text has checked its sources.
- Skipping simple retrieval or TF-IDF baselines.
