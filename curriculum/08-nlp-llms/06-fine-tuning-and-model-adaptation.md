# Fine-Tuning and Model Adaptation

## Learning objectives

Choose between prompting, retrieval, and fine-tuning; prepare a valid dataset; and evaluate adaptation without contaminating the test set.

## 1. Match the method to the problem

- **Prompting** changes task instructions or examples without changing model weights. It is fast to iterate and useful for formatting or simple task behavior.
- **Retrieval** supplies current, private, or auditable knowledge at inference time. It supports citations and updates without retraining.
- **Fine-tuning** changes model parameters using task examples. It can improve style, task patterns, or domain-specific response behavior, but it does not reliably install a database of facts or guarantee that learned facts will be recalled.

These methods can be combined. First establish a baseline and identify the failure source: missing knowledge suggests retrieval; inconsistent task format may suggest structured output/prompt changes; recurring behavior gaps may justify fine-tuning.

## 2. Training data quality

Use representative input-output pairs with clear labels and permission to train. Remove duplicates across train/validation/test. Split by source/user/time when examples are related. Include edge cases and desired refusals. Keep a separate evaluation set that is not repeatedly used to choose examples or hyperparameters.

For supervised fine-tuning (SFT), examples teach target responses. Preference optimization methods use comparisons or preferences to shape outputs; they require reliable preference data and their own evaluation. Poor labels or one-sided data can make behavior less useful or less safe.

## 3. Parameter-efficient methods

Full fine-tuning updates many model weights and can require substantial compute/storage. Parameter-efficient fine-tuning (PEFT), such as LoRA, learns a small set of additional parameters (often low-rank updates) while keeping most base weights frozen. It lowers resource requirements but still needs careful data, versioning, and evaluation. Quantization may reduce inference memory at some quality cost.

## 4. Evaluate the adaptation

Compare the adapted model with the original on the same held-out task suite. Measure task success, format compliance, factuality, refusal behavior, regression on general cases, latency, and resource use. Test for memorization and leakage, especially when training data are sensitive. Version the base model, training dataset, hyperparameters, adapter, and serving configuration.

## Practice

Suppose an assistant gives outdated policy answers. Decide whether prompt refinement, RAG, or fine-tuning is most appropriate. A strong answer usually favors retrieval for frequently changing, source-citable policy; define a case where fine-tuning might still help.

## Common mistakes

- Fine-tuning to teach changing facts that should be retrieved from an authoritative source.
- Measuring only training loss or a few cherry-picked outputs.
- Mixing near-duplicate records across evaluation splits.
- Forgetting that adapter, base model, tokenizer, and prompt versions must travel together.
