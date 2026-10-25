# LLM Foundations and Inference

## Learning objectives

Explain the flow from text to tokens to model output, predict which design choices affect latency/cost, and select a suitable model for a task.

## 1. What an LLM predicts

An autoregressive language model estimates a distribution over the next token conditioned on earlier tokens. Generation repeatedly selects a token and appends it to the context. A prompt may contain instructions, conversation history, retrieved evidence, and tool results, all of which consume context capacity.

Tokenization splits input into model-specific units. Token count is not the same as word count; code, numbers, rare terms, and some languages can use more tokens. Token IDs are representation indices, not semantic scores.

## 2. Context window and attention

The context window limits the tokens the model can process in a request (and sometimes output capacity is counted separately). Large context can carry more evidence but raises latency/cost and does not guarantee reliable use of every detail. Relevant, well-organized context usually beats indiscriminate dumping.

Transformers use attention to combine information across token positions. In autoregressive inference, a key/value cache can avoid recomputing earlier token representations during generation, trading memory for speed. Batching can improve accelerator utilization but may increase the wait time for an individual request.

## 3. Decoding controls

Greedy decoding picks the highest-probability next token. Temperature rescales logits before sampling; top-p restricts sampling to a probability mass. These affect diversity and repeatability, not factual verification. Use deterministic settings for stable formatting when appropriate, then validate the output separately.

## 4. Model choice is a system choice

Compare models on your task suite, not only general benchmarks. Consider quality, context requirements, languages, modalities, tool-call reliability, latency, throughput, privacy, cost, deployment region, and availability. A smaller model plus good retrieval may outperform a larger model for a narrow, grounded task.

For self-hosted models, consider accelerator memory, quantization quality, model licenses, batching, and serving stack. For hosted models, account for request limits, data handling terms, version changes, and provider outages.

## 5. Estimate request cost

At a high level, request cost depends on input tokens, output tokens, model pricing/compute, and repeated calls. Agent loops can make several model calls plus tools for one user request. Measure full-workflow cost per successful task, including retries and retrieval, not just the first model call.

## Practice

Take a long policy document and a set of realistic questions. Compare (a) sending the entire document and (b) retrieving a few relevant passages. Measure answer support, input tokens, and latency. Explain where the long-context approach or retrieval approach fails.

## Common mistakes

- Assuming a large context window means the model will attend to every included fact.
- Treating temperature as a truth or safety setting.
- Choosing models from a benchmark that does not reflect your users or constraints.
- Measuring only model-call latency while omitting retrieval, tools, and serialization.
