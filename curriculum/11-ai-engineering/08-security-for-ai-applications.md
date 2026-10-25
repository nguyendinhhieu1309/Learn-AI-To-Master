# Security for AI Applications

## Learning objectives

Threat-model an AI application, apply least privilege to tools and data, and test for vulnerabilities that arise from model behavior and system integration.

## 1. Threat model the system

List assets (data, credentials, model endpoint, user accounts), trust boundaries, actors, entry points, and plausible abuse. Consider the model, tools, retrieval sources, API, logs, worker/sandbox, and deployment account together. Identify what a malicious user or poisoned document can influence.

## 2. Protect tools and actions

Use server-side authorization and allowlists. Validate typed arguments and resource ownership immediately before execution. Separate read-only tools from side-effecting tools. Require approval for costly, irreversible, external, or high-impact actions. An LLM deciding to call a function is not authorization.

## 3. Protect data and credentials

Use secret managers, rotation, short-lived credentials, and minimal service permissions. Avoid sending confidential data to a model provider unless policy and user consent permit it. Redact logs, encrypt stored data, and define retention/deletion. Secure vector stores and caches with tenant-aware filtering.

## 4. Prompt injection and indirect control

User input and retrieved content may contain hostile instructions. Delimit content and instruct the model to treat it as data, but enforce security in code: prevent untrusted text from changing permissions, calling arbitrary tools, or exposing secrets. Test direct and indirect injection paths and inspect resulting traces.

## 5. Supply chain and sandboxing

Pin/scan dependencies, verify model artifacts, review skills/scripts and MCP tool servers before enabling them, and isolate code execution. A Python function or skill that runs a shell command has the authority of its runtime unless restricted. Use filesystem/network/CPU/time limits in a sandbox.

## 6. Security testing

Test cross-tenant data access, malformed payloads, oversized files, prompt injection, tool argument abuse, replayed requests, rate limits, secret leakage, and denial-of-service patterns. Report findings by exploitability and impact. Keep red-team data separate from production data.

## Practice

Threat-model a RAG agent with a `search_docs` read tool and `send_email` action. Describe an attack through a malicious retrieved document, then define code-level controls, human approval conditions, logging/privacy policy, and tests that demonstrate the boundary.

## Common mistakes

- Believing a system prompt is an access-control boundary.
- Giving the model shell, SQL, or filesystem access when narrow functions suffice.
- Reviewing only prompt text while ignoring dependencies and tool-server code.
- Treating a sandbox as secure without checking its network, file, and credential permissions.
