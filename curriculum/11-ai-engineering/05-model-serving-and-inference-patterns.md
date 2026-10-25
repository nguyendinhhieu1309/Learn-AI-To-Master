# Model Serving and Inference Patterns

## Learning objectives

Select an inference pattern, define a model-serving contract, and measure the full request path under realistic load.

## 1. Batch, synchronous, and asynchronous work

- **Batch inference:** process a known dataset or queue in bulk. Efficient throughput matters; results can arrive later.
- **Synchronous online inference:** caller waits for a response. Latency, availability, and request limits matter.
- **Asynchronous job:** API returns a job ID; work continues in a queue and the caller polls/subscribes for completion. Useful for long jobs, large files, or human review.
- **Streaming response:** return partial tokens/events as they are produced. It improves perceived time-to-first-result but requires cancellation, buffering, and partial-error semantics.

Choose based on user experience and operational constraints. Do not expose a synchronous endpoint for work that routinely exceeds caller timeouts.

## 2. The serving path

An inference request may pass through authentication → validation → preprocessing/tokenization → queue → model runtime → postprocessing → output validation → response/logging. Measure each stage. A slow database lookup or image decode can dominate model latency.

Define input/output schema, model/preprocessing version, units, error cases, timeout, maximum payload, and response semantics. Include a request/correlation ID so logs and traces can be joined without exposing raw sensitive content.

## 3. Throughput and latency

Latency is time per request; throughput is requests or examples per unit time. Batching can improve accelerator utilization by processing several inputs together, but it may delay an early request while a batch fills. Measure p50, p95, and p99 latency under expected concurrency, not just one local call.

Capacity planning considers model memory, accelerator memory, token/image sizes, batch size, concurrency, queue length, and warm-up. For LLMs, input length and generated token count strongly affect time and cost. For vision, image resolution affects compute and memory.

## 4. Scale and optimize in order

Start with a correct baseline service. Profile before optimizing. Options include caching safe deterministic results, batching, smaller models, quantization, optimized runtimes, request routing, and horizontal scaling. Caches need keys that include relevant model, prompt, preprocessing, tenant/permission, and data versions; never serve one user's private result to another.

## 5. Backpressure and graceful failure

Use bounded queues, request limits, timeouts, cancellation, and load shedding so overload does not cascade. Define fallback behavior: return a partial result, queue the job, ask the user to retry, use a simpler model, or route to a human. A fallback should be explicit and observable.

## Practice

Design an image classification API for bursts of 100 requests/sec. State whether it is synchronous or asynchronous, max image size, queue policy, p95 latency objective, scaling signal, timeout, and behavior when GPU capacity is exhausted.

## Common mistakes

- Measuring only model execution time rather than end-to-end latency.
- Scaling replicas without checking memory, queueing, or provider quotas.
- Retrying indefinitely during overload.
- Caching predictions without including model/data/permission version in the cache key.
