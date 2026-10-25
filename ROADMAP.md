# Learning Roadmap

This roadmap is self-paced. Move forward when you can explain the ideas in your own words and complete the checkpoint without copying a solution. Revisit earlier topics whenever a later lesson exposes a gap.

## Core path

### 0. Set up your learning workflow

Read [How to use this path](curriculum/00-start-here/01-how-to-use-this-path.md). Set up Python, a code editor or notebook environment, Git, and a place to keep notes.

**Checkpoint:** Run a short Python script, save your notes in Markdown, and make a small Git commit.

### 1. Build the math you need

Study [Mathematics for AI](curriculum/01-math/README.md) alongside programming. Focus first on vectors, matrices, functions, derivatives, probability, and summary statistics.

**Checkpoint:** Explain a dot product, a derivative, a mean, and a probability using a small numerical example.

### 2. Learn Python and software engineering

Follow [Python and computing](curriculum/02-python/README.md). Learn project structure, environments, Git, testing, APIs, and data contracts in [AI Engineering](curriculum/11-ai-engineering/README.md).

**Checkpoint:** Package a small Python program that reads a CSV file, computes summaries, validates input, logs useful context, and has clear run instructions.

### 3. Learn to work with data

Use [Data processing and engineering](curriculum/03-data/README.md) to go from raw tables to model-ready datasets and repeatable ingestion and feature pipelines. Practice with tabular data before moving to text, images, time series, or 3D data.

**Checkpoint:** Create a reproducible data-cleaning workflow, document its assumptions, and check for data leakage.

### 4. Learn classical machine learning

Follow [Machine learning](curriculum/04-machine-learning/README.md). Compare simple baselines with linear models, trees, and unsupervised methods. Learn how metrics and data splits affect conclusions.

**Checkpoint:** Train at least two models, evaluate them on held-out data, and explain where each model fails.

### 5. Learn deep learning

Continue with [Deep learning](curriculum/05-deep-learning/README.md). Understand tensors, loss functions, gradient descent, and training diagnostics before specializing.

**Checkpoint:** Train a small neural network and inspect its learning curves, errors, and reproducibility settings.

## Choose a specialization

Once the core path is comfortable, choose one route first. You can return for another later.

- **Vision and robotics:** [Computer vision](curriculum/06-computer-vision/README.md) → [3D vision and robotics](curriculum/07-3d-vision-robotics/README.md) → [AI Engineering](curriculum/11-ai-engineering/README.md).
- **Language and agents:** [NLP, LLMs, and generative AI](curriculum/08-nlp-llms/README.md) → [Agentic AI](curriculum/10-agentic-ai/README.md) → evaluation and deployment.
- **ML/data engineering:** [Data processing and engineering](curriculum/03-data/README.md) → [Machine learning](curriculum/04-machine-learning/README.md) → [AI Engineering](curriculum/11-ai-engineering/README.md).
- **Decision-making:** [Reinforcement learning](curriculum/09-reinforcement-learning/README.md) → a simulation project → evaluation and safe deployment.

Across all routes, study [Responsible AI](curriculum/12-responsible-ai/README.md) to examine who benefits, who may be harmed, what data use is appropriate, and how people can review or challenge system outcomes.

## Suggested project sequence

1. **Software foundation:** package a small Python data tool with a command-line or API interface.
2. **Data report:** build a repeatable ingestion/cleaning pipeline and explain its quality issues and limitations.
3. **Prediction baseline:** compare a simple rule, a linear model, and a tree-based model with a sound validation strategy.
4. **Deep learning application:** train or adapt an image/text model, analyze errors, and compare against a simpler baseline.
5. **LLM application:** build a grounded retrieval system, evaluate retrieval and answers separately, and include unanswerable cases.
6. **Agent system:** build a bounded agent with a reusable skill, narrow tools, a harness, traces, and human approval for a consequential action.
7. **Reliable delivery:** deploy one project with observability, cost controls, privacy/security notes, and rollback.

Use the [Core AI Engineer Capstone](projects/CORE_AI_ENGINEER_CAPSTONE.md) as the detailed implementation and review guide for the first four stages and final delivery.

## Study rhythm

For each lesson: read, reproduce the example, solve the exercise without looking at a solution, then write a short summary. Use the [cheat sheet](CHEATSHEET.md) for recall after you understand the derivation. For each project: define the problem and metric first, keep a baseline, track changes, and present both successful and failed experiments.

## AI Engineer readiness checklist

You are ready to take on a small end-to-end AI project when you can:

- implement and explain the data path from source to prediction;
- make a valid split and defend the metric and baseline;
- package code so another developer can reproduce it;
- expose model behavior behind a clear input/output contract;
- diagnose model, data, software, and infrastructure failures separately;
- monitor latency, errors, cost, data changes, and task quality;
- state privacy, security, fairness, and deployment limitations.
