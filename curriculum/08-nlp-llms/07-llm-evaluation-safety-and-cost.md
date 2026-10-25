# LLM Evaluation, Safety, and Cost

## Learning objectives

Evaluate an LLM application as a complete system, combine automated and human review, and manage quality, risk, and operating cost.

## 1. Build a task-specific evaluation set

Write examples from the intended users and failure modes before comparing models. Include normal, edge, ambiguous, unanswerable, multilingual, and adversarial cases. Store expected outcomes, acceptable alternatives, evidence requirements, and severity. Keep development examples separate from a final holdout set.

## 2. Score multiple dimensions

Depending on the task, measure correctness, completeness, groundedness, citation support, schema validity, refusal quality, tool-call accuracy, robustness, latency, cost, and privacy failures. Report error categories and examples alongside averages. A model can have a strong average and still fail catastrophically on an important edge case.

## 3. Human review and LLM judges

Human review is useful for ambiguity, preference, and high-impact errors. Write a rubric, train reviewers on examples, and measure agreement where possible. An LLM judge can scale a first pass but may favor longer answers, share the same blind spots as the generator, or be sensitive to phrasing. Calibrate it against expert labels and keep human audits.

## 4. Test the whole system

Evaluate extraction, retrieval, generation, tools, permissions, retries, and final user experience. Test injected instructions inside sources, malformed output, unavailable dependencies, low-confidence predictions, and permission boundaries. Check the outcome in the environment; a transcript claiming success is not proof that an action occurred.

## 5. Quality-cost trade-offs

Use the smallest/least costly model that meets requirements. Consider prompt caching, shorter irrelevant context, retrieval, structured outputs, bounded retries, and routing easier requests to a smaller model. Measure cost per successful task and tail latency, not only cost per token or one successful call.

## 6. Safety and privacy

Minimize retained prompts, redact sensitive fields, restrict trace access, and define deletion schedules. Screen untrusted inputs where appropriate, but combine model checks with deterministic validation, authorization, and human oversight for consequential actions. See [Responsible AI](../12-responsible-ai/README.md).

## Practice

Create an evaluation report for a document-answering assistant with 30 cases. Separate retrieval misses, unsupported claims, invalid citations, refusal mistakes, and latency/cost. Add a release threshold for critical failure categories and a human escalation rule.

## Common mistakes

- Treating a single LLM-judge score as objective truth.
- Evaluating only generated text, not permissions, retrieved evidence, or real side effects.
- Improving average quality while allowing a severe error class to increase.
- Logging raw sensitive inputs because they are convenient for debugging.
