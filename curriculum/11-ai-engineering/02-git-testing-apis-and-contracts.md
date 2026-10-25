# Git, Testing, APIs, and Data Contracts

## Learning objectives

Review changes with Git, choose appropriate tests for AI software, and expose model behavior through a clear interface.

## 1. Git as a work history

Git records snapshots (commits) and lets a team review focused changes. Check `git status` and inspect a diff before committing. Write a commit message that explains the change. Branches and pull requests support parallel work and review; never commit raw credentials or restricted datasets.

## 2. Testing layers

- **Unit tests:** deterministic functions such as date parsing, feature transforms, threshold logic, and permission checks.
- **Contract/schema tests:** verify required columns, types, units, and output shapes.
- **Integration tests:** check that storage, model loading, retrieval, or a tool adapter connects correctly.
- **End-to-end tests:** run a representative task through the full application.
- **Model evaluations:** measure quality on a versioned dataset and task suite; these are statistical evidence, not a replacement for software tests.

Model tests should use representative cases and tolerances when exact floating-point outputs vary. Assert important properties (shape, valid range, latency budget, or acceptable score change), not a brittle exact decimal for every neural output.

## 3. HTTP API basics

HTTP requests contain a method, path, headers, and optional body. A prediction API typically accepts JSON, validates it, performs preprocessing/inference, and returns JSON plus an appropriate status. `POST` is commonly used for a prediction body; `GET` is often used to retrieve a resource. Use explicit error responses for invalid input, unavailable model, timeout, or internal failure. Do not return a successful prediction when the model did not run.

Define a versioned contract: fields, type, units, category vocabulary, optional values, output label semantics, confidence meaning, and error behavior. Backward compatibility matters when clients deploy independently.

## 4. Retries, timeouts, and idempotency

Network calls can fail or time out. Add a timeout to external calls; retry only when appropriate, with backoff and a retry limit. A non-idempotent action (such as placing an order) can be repeated accidentally if a timeout occurs after the action succeeded. Use an idempotency key and verify action status before retrying.

## 5. Security at the interface

Authenticate clients, authorize each operation, rate-limit abuse, validate payload sizes and fields, and avoid returning stack traces to untrusted callers. Input validation reduces malformed data; it does not make model outputs correct. For LLM-based endpoints, also account for prompt injection, output validation, and untrusted retrieved content.

## Practice

Write an API contract for an image classifier: accepted formats, maximum dimensions/bytes, preprocessing, output classes, confidence semantics, timeouts, and error codes. Design one unit test, one contract test, and one end-to-end evaluation case.

## Common mistakes

- Treating one test accuracy assertion as complete model evaluation.
- Retrying an irreversible action without an idempotency strategy.
- Returning internal secrets or exception traces to the caller.
- Changing an output label mapping without a contract version change.
