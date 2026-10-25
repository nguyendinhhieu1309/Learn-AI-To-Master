# Prompt Design and Structured Outputs

## Learning objectives

Write prompts with explicit task boundaries, separate trusted instructions from untrusted data, and validate machine-readable model outputs.

## 1. A prompt is part of the program

A useful prompt usually states: task, input meaning, constraints, output format, and what to do when evidence is missing. Provide a small number of representative examples when they clarify edge cases. Keep system/developer instructions separate from user-provided text and retrieved content.

Avoid vague demands such as “be perfect.” Replace them with observable requirements: include source IDs, return a fixed schema, abstain if required fields are absent, or ask one clarifying question.

## 2. Delimit data, but still distrust it

Mark where a document or user payload begins and ends and tell the model to treat it as data. Delimiters make boundaries clearer, but they are not an authorization mechanism. A malicious document can still contain text that tries to redirect the model; enforce permissions and tool rules in application code.

## 3. Structured output

For downstream code, ask for schema-constrained output (such as JSON or a typed model) when the provider supports it. Then parse and validate it. Schema validity only guarantees shape/types; it does not guarantee factual correctness or allowed actions.

```python
from pydantic import BaseModel, Field

class TicketSummary(BaseModel):
    category: str = Field(description="One of billing, technical, or other")
    needs_human_review: bool
    summary: str
```

After receiving output, validate `category` against an allowlist and apply authorization logic before taking action. Return a recoverable validation error or request regeneration with a bounded retry; do not silently invent missing fields.

## 4. Prompt injection and trust boundaries

Prompt injection is an attempt to influence model behavior through untrusted text. Treat user input, retrieved pages, email bodies, code, and tool outputs as data. Keep secrets out of the prompt unless strictly required; do not give the model a broad tool because an instruction says it needs one. The tool executor must independently check identity, permission, and arguments.

## 5. Version and test prompts

Store prompts with versions and evaluate them against a fixed set of typical, ambiguous, unanswerable, and adversarial cases. Compare output properties, not just a handpicked example. A prompt edit can affect tool choice, refusal, length, and latency; regression-test each behavior relevant to the application.

## Practice

Write an extraction prompt for invoice totals. Specify currency, tax treatment, output fields, and behavior if two totals conflict. Create tests for missing currency, multiple pages, OCR error, and an injected instruction in the invoice. Decide which checks belong in code rather than the prompt.

## Common mistakes

- Assuming JSON output means the content is correct.
- Placing a user document after trusted instructions without marking it as untrusted data.
- Relying on prompt language to enforce access control.
- Changing prompts in production without versioning or regression evaluation.
