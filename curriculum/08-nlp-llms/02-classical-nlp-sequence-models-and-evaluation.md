# Classical NLP, Sequence Models, and Evaluation

Modern language models build on ideas from classical NLP. Understanding tokenization, count-based models, sequence labeling, and evaluation helps diagnose failures and select the right tool for a task.

## 1. Text is not a neutral sequence of words

Tokenization choices affect vocabulary size, rare words, morphology, punctuation, and languages without whitespace. A word tokenizer, character tokenizer, and subword tokenizer produce different inputs. Normalize only what the task allows: lowercasing can discard useful case signals, and aggressive stemming can erase meaning.

For every text dataset, document encoding, language, normalization, truncation, missing text handling, and label policy. Split by author, conversation, document source, or time when those factors could leak between train and test.

## 2. Bag-of-words and TF-IDF

Bag-of-words represents a document by token counts and ignores token order. TF-IDF downweights words common across documents and emphasizes words informative for a particular document. It is an efficient, strong baseline for classification and retrieval.

Fit vocabulary and inverse-document-frequency statistics on training data only. Use n-grams when local word order matters, and control vocabulary size to manage memory and rare noisy features. Linear models such as logistic regression or linear SVM often work well on these sparse vectors.

## 3. Language modeling

A language model estimates a sequence probability via the chain rule:

$$
P(w_1,\ldots,w_T)=\prod_{t=1}^{T}P(w_t\mid w_1,\ldots,w_{t-1})
$$

An n-gram model approximates the context using the previous $n-1$ tokens. Sparse counts require smoothing so unseen sequences do not get zero probability. Neural language models learn distributed representations and can use longer context, but still depend on tokenization, training data, and objective.

Perplexity is the exponentiated average negative log-likelihood:

$$
\operatorname{PPL} = \exp\left(-\frac{1}{T}\sum_{t=1}^{T}\log P(w_t\mid w_{<t})\right)
$$

It measures predictive fit on a tokenized test set. Perplexity values are not directly comparable across different tokenizers, datasets, or context conventions.

## 4. Sequence labeling: HMM and CRF intuition

Named entity recognition, part-of-speech tagging, and chunking assign a label to each token. A hidden Markov model makes simplifying assumptions about transitions and emissions. A conditional random field models a globally normalized sequence score and can enforce dependencies among neighboring labels, such as valid entity-tag transitions.

Modern transformer encoders often replace classical sequence models, but the structured-prediction view remains useful: labels are not always independent decisions, and entity boundaries need consistent decoding.

## 5. Evaluation for NLP tasks

- Classification: per-class precision/recall/F1, confusion matrix, and calibration where probabilities are used.
- Named entities: state exact-match versus token-level scoring, and define span normalization.
- Machine translation: BLEU is n-gram overlap and can miss semantic equivalence; inspect human-rated quality and error categories.
- Summarization: ROUGE measures overlap and may reward copied phrases; factuality and coverage need separate evaluation.
- Retrieval: recall@k, mean reciprocal rank, or nDCG at the operating cutoff.

Use representative human review. Annotation guidelines and adjudication matter because multiple valid phrasings can express the same answer. For generative systems, score factual correctness, instruction following, safety, and usefulness separately instead of collapsing them into one opaque number.

## 6. Data leakage in text

Near-duplicate documents, template text, author names, conversation turns, and future responses can leak labels. Search for duplicates across splits and inspect high-weight n-grams. A model that recognizes a source or template may perform well on a random split but fail on new sources.

## 7. Exercises

1. Why can TF-IDF plus a linear classifier be a useful baseline even when a transformer is available?
2. A language model has lower perplexity after switching to a tokenizer with twice as many tokens. Is it necessarily better?
3. A summarizer achieves high ROUGE but invents dates. Which additional evaluation is needed?
4. Sentiment data contains many product reviews copied from templates. How would you construct a more realistic split?

## References

- [NLTK Book](https://www.nltk.org/book/)
- [scikit-learn: Working with text data](https://scikit-learn.org/stable/tutorial/text_analytics/working_with_text_data.html)
- [SacreBLEU](https://github.com/mjpost/sacrebleu)
