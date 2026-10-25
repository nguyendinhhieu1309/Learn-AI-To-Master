<h1 align="center">
  <img src="assets/logo.svg" alt="Learn AI to Master" width="900">
</h1>

<p align="center"><a href="README.md">English</a> · <a href="translations/vi/README.md">Tiếng Việt</a></p>

<p align="center">A free, practical learning path from AI foundations to modern machine learning, computer vision, robotics, reinforcement learning, and agentic systems.</p>

<p align="center"><strong>Author:</strong> Nguyen Dinh Hieu</p>

<p align="center"><a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-147D79?style=flat-square" alt="MIT License"></a></p>

The English edition is the source curriculum. A Vietnamese edition is available under [translations/vi](translations/vi/README.md); technical terms remain in English where that makes learning and search easier. Lessons are Markdown-first and designed to work with coding exercises and projects.

> **Project status:** The repository includes the English source curriculum and a Vietnamese mirror with guided lessons, worked examples, practice, projects, and cheat sheets. The curriculum can continue to grow through deeper exercises and reviewed project walkthroughs.

## Who this is for

- Beginners who want a clear sequence instead of a pile of disconnected links.
- Programmers who want to understand the ideas behind AI systems.
- Learners interested in a practical specialization such as computer vision, robotics, language models, or reinforcement learning.

You do not need to know advanced mathematics before starting. The path introduces the required math as it becomes useful. Basic computer use and a willingness to write Python are helpful.

## Start here

1. Read [How to use this learning path](curriculum/00-start-here/01-how-to-use-this-path.md).
2. Follow the [curriculum map](curriculum/README.md) and [roadmap](ROADMAP.md).
3. Start with [Python and computing foundations](curriculum/02-python/README.md), while reviewing math topics as needed.
4. Keep notes, run the exercises yourself, and publish small projects as you go.

For a quick reminder of formulas, metrics, shapes, and workflows, use the [AI learning cheat sheet](CHEATSHEET.md). Learn the linked lessons first; the cheat sheet is a compact reference, not a replacement for examples and practice.

## Curriculum at a glance

| Stage | Subject | What you will be able to do |
| --- | --- | --- |
| 0 | [Start here](curriculum/00-start-here/README.md) | Set goals, set up a learning workflow, and navigate the material. |
| 1 | [Python and computing](curriculum/02-python/README.md) | Write readable Python and use the tools common in AI work. |
| 2 | [Mathematics for AI](curriculum/01-math/README.md) | Use linear algebra, calculus, probability, statistics, and optimization. |
| 3 | [Data processing and engineering](curriculum/03-data/README.md) | Clean, validate, query, and build reliable batch, streaming, and feature pipelines. |
| 4 | [Machine learning](curriculum/04-machine-learning/README.md) | Train, evaluate, and explain classical ML models. |
| 5 | [Deep learning](curriculum/05-deep-learning/README.md) | Build neural networks and understand their training process. |
| 6 | [Computer vision](curriculum/06-computer-vision/README.md) | Work with images and video, from classical methods to deep models. |
| 7 | [3D vision and robotics](curriculum/07-3d-vision-robotics/README.md) | Connect cameras, geometry, point clouds, ROS 2, and robot perception. |
| 8 | [NLP, LLMs, and generative AI](curriculum/08-nlp-llms/README.md) | Learn text and transformers, then build and evaluate LLM applications. |
| 9 | [Reinforcement learning](curriculum/09-reinforcement-learning/README.md) | Formulate sequential decisions and train agents with feedback. |
| 10 | [Agentic AI](curriculum/10-agentic-ai/README.md) | Build skill-aware, tool-using, stateful workflows with frameworks and runtimes. |
| 11 | [AI engineering](curriculum/11-ai-engineering/README.md) | Build maintainable software, deploy and monitor models, and operate reliable AI systems. |
| 12 | [Responsible AI](curriculum/12-responsible-ai/README.md) | Assess fairness, privacy, transparency, misuse risks, and human oversight. |

Stages 0–5 form the shared foundation. After that, choose a specialization based on your goals; you do not have to study every advanced topic in a single straight line. Study responsible AI alongside every stage. AI Engineering supports every specialization.

## Repository layout

```text
.
├── .gitignore
├── README.md
├── CHEATSHEET.md
├── ROADMAP.md
├── LICENSE
├── assets/
│   └── logo.svg
├── curriculum/
│   ├── 00-start-here/
│   ├── 01-math/
│   ├── 02-python/
│   ├── 03-data/
│   ├── 04-machine-learning/
│   ├── 05-deep-learning/
│   ├── 06-computer-vision/
│   ├── 07-3d-vision-robotics/
│   ├── 08-nlp-llms/
│   ├── 09-reinforcement-learning/
│   ├── 10-agentic-ai/
│   ├── 11-ai-engineering/
│   ├── 12-responsible-ai/
│   └── README.md
├── projects/
└── translations/
    ├── README.md
    └── vi/
```

## Learning approach

Each subject is organized as a sequence of short lessons. A mature lesson should explain the idea, show a worked example, include a hands-on exercise, and point out common mistakes. Each module ends with a project or checkpoint that combines its main skills.

The curriculum favors understanding and reproducibility: use small datasets first, establish a simple baseline, keep a validation set separate, record experiments, and explain what your results do and do not show.

## Deeper lessons and formulas

See the [curriculum map](curriculum/README.md) for expanded lessons grouped by subject, and the [AI mathematics formula cheatsheet](curriculum/01-math/FORMULA_CHEATSHEET.md) for a concise formula reference.

## License

This repository is released under the [MIT License](LICENSE).
