# Inference Serving, Latency, and Cost

Model quality is only one part of an AI system. A useful service must meet its latency, throughput, memory, availability, and cost requirements under realistic traffic.

## 1. Define the service objective

Specify request size, traffic shape, concurrency, hardware, quality constraints, and latency objective. Report percentile latency, such as p50, p95, and p99; an average hides tail behavior. Distinguish queueing, preprocessing, model execution, tool or retrieval calls, serialization, and network time.

For autoregressive generation, time-to-first-token and token generation rate are separate measures. For batch inference, throughput and end-to-end completion time matter. The correct measure depends on user experience.

## 2. Amdahl’s law and bottlenecks

If fraction $p$ of work is accelerated by factor $s$, total speedup is:

$$
S = \frac{1}{(1-p)+p/s}
$$

Speeding up the neural network has little impact if retrieval, database access, CPU preprocessing, or queueing dominates. Profile stage-by-stage before optimizing.

## 3. Batching and concurrency

Batching amortizes fixed compute overhead and can improve accelerator utilization, but increases queueing delay and may waste work for variable-length requests. Dynamic batching needs a maximum wait time and maximum batch size. For language models, continuous batching can make use of devices while sequences start and finish at different times.

Bound concurrency and queues. An unbounded queue turns overload into extreme latency and memory use. Apply backpressure, deadlines, admission control, and clear overload responses.

## 4. Model and request optimizations

- Quantization reduces weight precision and memory use; validate task quality and hardware support.
- Distillation transfers behavior to a smaller model; evaluate the student on task-critical slices.
- Pruning can reduce parameters, though speed gains depend on sparse-kernel support.
- Caching helps repeated requests, but cache keys, freshness, privacy boundaries, and invalidation must be correct.
- Speculative decoding can improve token generation speed when a smaller draft model is accepted often enough.

Measure quality after each optimization; the best compression ratio is not useful if it breaks rare but important cases.

## 5. Capacity and cost estimation

Estimate peak requests per second, tokens or examples per request, concurrency, memory per model replica, accelerator utilization, and redundancy for failures. Include model load time, warm-up, and autoscaling delay. For language models, input and output token counts both affect cost and latency.

Compare cost per successful task, not only cost per request. A cheaper model that needs repeated retries or human correction can be more expensive overall.

## 6. Reliability and safe degradation

Define timeouts and retry policy by operation. Retrying non-idempotent tool actions can duplicate effects. Use circuit breakers or fallbacks when dependencies fail; the fallback should preserve a safe behavior such as abstention or human escalation. Do not let a fallback silently bypass access controls or validation.

Test cold starts, dependency outages, model load failures, malformed responses, resource exhaustion, and partial streaming failures. Maintain a rollback path and a known-good artifact.

## 7. Benchmarking correctly

Warm up the runtime, pin model and hardware versions, use representative inputs and output lengths, test realistic concurrency, and report tail latency with throughput. Separate prefill and decode for generative models. Do not compare numbers measured under different batching, quantization, context length, or hardware settings without stating those differences.

## 8. Exercises

1. Network inference is 20% of request time and becomes four times faster. What is the maximum overall speedup under Amdahl’s law?
2. Batching increases throughput but violates p99 latency. Which controls could balance the objective?
3. Quantization cuts serving cost by 40%, but harms a rare safety-critical slice. What should promotion require?
4. A tool request times out and the service retries automatically. What property must hold to avoid duplicate side effects?

## References

- [NVIDIA Triton Inference Server documentation](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/)
- [vLLM documentation](https://docs.vllm.ai/)
- [Google SRE Book: Handling overload](https://sre.google/sre-book/handling-overload/)
