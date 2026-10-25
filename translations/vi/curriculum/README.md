# Bản đồ Curriculum

Curriculum đi từ nền tảng chung tới ứng dụng chuyên sâu. Mỗi trang môn học liệt kê learning objective, kiến thức tiên quyết, thứ tự bài gợi ý và project checkpoint. Khóa học tự học theo tốc độ cá nhân; thứ tự là gợi ý, không phải lịch cứng.

~~~mermaid
flowchart LR
    Python[Python] --> Eng[AI Engineering]
    Math[Toán và thống kê] --> ML[Machine learning]
    Eng --> Data[Xử lý dữ liệu và data engineering]
    Data --> ML
    ML --> DL[Deep learning]
    DL --> CV[Computer vision]
    CV --> Robot[3D vision và robotics]
    DL --> NLP[NLP, LLM và AI tạo sinh]
    NLP --> Agent[Agentic AI]
    ML --> Eng
    NLP --> Eng
    Agent --> Eng
    Responsible[Responsible AI] -. xuyên suốt .-> ML
~~~

## Nền tảng chung

1. [Bắt đầu tại đây](00-start-here/README.md)
2. [Python và computing](02-python/README.md)
3. [Toán học cho AI](01-math/README.md)
4. [Xử lý và data engineering](03-data/README.md)
5. [Machine learning](04-machine-learning/README.md)
6. [Deep learning](05-deep-learning/README.md)

Có thể học toán song song với Python. Học xử lý dữ liệu trước model training để hiểu data quality, cách chia tập và leakage ngay từ đầu.

## Chuyên ngành và thực hành hỗ trợ

- [Computer vision](06-computer-vision/README.md) → [3D vision và robotics](07-3d-vision-robotics/README.md)
- [NLP, LLM và AI tạo sinh](08-nlp-llms/README.md) → [Agentic AI](10-agentic-ai/README.md)
- [Reinforcement learning](09-reinforcement-learning/README.md)
- [AI Engineering](11-ai-engineering/README.md) áp dụng xuyên các chuyên ngành.
- [Responsible AI](12-responsible-ai/README.md) cần được cân nhắc trong toàn bộ curriculum.

## Một module học đầy đủ cần có gì?

Bản curriculum tiếng Việt bao gồm bài học có hướng dẫn, ví dụ giải từng bước, bài thực hành, failure mode thường gặp và cách nối kiến thức với project. Các module có thể tiếp tục mở rộng bằng lab code và capstone được review.

## Chỉ mục bài học mở rộng

Tất cả bài nằm trong thư mục môn học tương ứng; chương nâng cao không tạo thêm thư mục khóa học riêng.

- **Toán:** [calculus và gradient hồi quy tuyến tính](01-math/02-calculus-and-optimization.md), [statistical inference](01-math/04-statistical-inference-and-experiment-design.md), [linear algebra, least squares, PCA và SVD](01-math/05-linear-algebra-for-machine-learning.md), [information theory và numerical method](01-math/06-information-theory-and-numerical-computation.md), [phân phối xác suất và Bayes](01-math/07-probability-distributions-and-bayes.md), [likelihood, MLE và MAP](01-math/08-likelihood-mle-and-map.md), [convex optimization](01-math/09-convex-optimization-and-constraints.md), [formula cheatsheet](01-math/FORMULA_CHEATSHEET.md).
- **Python và data:** [Python foundations, OOP và DSA](02-python/README.md); [ngôn ngữ Python cốt lõi](02-python/04-python-core-language.md); [OOP](02-python/06-object-oriented-programming.md); [complexity](02-python/07-recursion-and-complexity.md); [linked list](02-python/09-linked-lists-stacks-and-queues.md); [BST và heap](02-python/11-trees-bst-and-heaps.md); [graph](02-python/12-graphs-and-graph-algorithms.md); tiếp theo [feature engineering](03-data/04-data-cleaning-eda-and-feature-engineering.md), [SQL và tạo dataset](03-data/05-sql-and-dataset-construction.md), [ETL/ELT](03-data/07-etl-elt-pipelines-and-orchestration.md), [data quality và feature store](03-data/09-data-quality-feature-stores-and-streaming.md).
- **ML và DL:** [ML validation](04-machine-learning/04-generalization-validation-and-model-selection.md), [linear regression](04-machine-learning/07-linear-regression-and-gradient-descent.md), [logistic regression và softmax](04-machine-learning/08-logistic-regression-perceptron-and-softmax.md), [k-NN và k-means](04-machine-learning/09-knn-and-k-means.md), [SVM](04-machine-learning/10-support-vector-machines.md), [PCA/SVD](04-machine-learning/11-pca-svd-and-dimensionality-reduction.md), [tree và Naive Bayes](04-machine-learning/12-decision-trees-and-naive-bayes.md), [recommendation system](04-machine-learning/13-recommendation-systems.md).
  Deep-learning derivation gồm [backpropagation](05-deep-learning/05-backpropagation-worked-example.md), [CNN convolution](05-deep-learning/06-convolution-and-cnn-shapes.md), [transformer attention](05-deep-learning/07-attention-and-transformers-by-hand.md), [generative model](05-deep-learning/08-generative-models-foundations.md), [training](05-deep-learning/03-training-deep-networks-in-practice.md) và [network design/loss](05-deep-learning/04-neural-network-design-losses-and-diagnostics.md).
- **Vision và robotics:** [CV evaluation](06-computer-vision/04-detection-segmentation-and-error-analysis.md), [image pipeline](06-computer-vision/05-image-pipelines-and-vision-training-workflow.md), [stereo và SLAM](07-3d-vision-robotics/04-stereo-visual-odometry-slam-and-point-clouds.md), [point-cloud learning](07-3d-vision-robotics/05-point-cloud-processing-and-deep-learning.md).
- **NLP, LLM và agent:** [classical NLP](08-nlp-llms/02-classical-nlp-sequence-models-and-evaluation.md), [LLM foundation](08-nlp-llms/03-llm-foundations-and-inference.md), [prompting](08-nlp-llms/04-prompt-design-and-structured-outputs.md), [retrieval](08-nlp-llms/05-retrieval-engineering.md), [fine-tuning](08-nlp-llms/06-fine-tuning-and-model-adaptation.md), [LLM evaluation](08-nlp-llms/07-llm-evaluation-safety-and-cost.md), [RAG evaluation](08-nlp-llms/08-rag-evaluation-and-safety.md), [RAG observability](08-nlp-llms/09-rag-evaluation-and-observability.md), [RAG system lab](08-nlp-llms/10-rag-system-worked-lab.md), [retrieval pattern nâng cao](08-nlp-llms/11-advanced-rag-retrieval-patterns.md) và [policy-gradient RL](09-reinforcement-learning/03-policy-gradients-actor-critic-and-practice.md).
- **AI Engineering:** toàn bộ software engineering, MLOps và AI systems nằm trong [một module](11-ai-engineering/README.md), gồm [model lifecycle](11-ai-engineering/04-model-lifecycle-testing-and-monitoring.md), [inference latency/cost](11-ai-engineering/09-inference-serving-latency-and-cost.md) và [LLMOps evaluation/experiment design](11-ai-engineering/11-llmops-evaluation-and-experiment-design.md).

## Workbook và capstone có hướng dẫn

- [Bộ bài toán toán học có lời giải](01-math/PROBLEM_SET_WITH_SOLUTIONS.md)
- [Python lab tái lập](02-python/03-reproducible-python-lab.md)
- [Bài tập Python, OOP và DSA có lời giải](02-python/PROBLEM_SET_WITH_SOLUTIONS.md)
- [Workshop data validation và leakage](03-data/10-validation-and-leakage-workshop.md)
- [Bộ bài tập data engineering có lời giải](03-data/PROBLEM_SET_WITH_SOLUTIONS.md)
- [Ensemble và boosting](04-machine-learning/14-ensembles-and-boosting.md)
- [Calibration, decision và uncertainty](04-machine-learning/15-calibration-and-uncertainty.md)
- [Bộ bài tập machine learning có lời giải](04-machine-learning/PROBLEM_SET_WITH_SOLUTIONS.md)
- [Bài deep learning về optimizer/training dynamics](05-deep-learning/09-optimizers-and-training-dynamics.md)
- [Lab train tiny classifier](05-deep-learning/10-tiny-classifier-training-lab.md)
- [Bộ bài tập deep learning có lời giải](05-deep-learning/PROBLEM_SET_WITH_SOLUTIONS.md)
- [Computer vision worked lab](06-computer-vision/06-computer-vision-worked-lab.md)
- [Bộ bài tập computer vision có lời giải](06-computer-vision/PROBLEM_SET_WITH_SOLUTIONS.md)
- [Point-cloud registration lab](07-3d-vision-robotics/06-point-cloud-registration-lab.md)
- [Bộ bài tập 3D vision/robotics có lời giải](07-3d-vision-robotics/PROBLEM_SET_WITH_SOLUTIONS.md)
- [RAG system worked lab](08-nlp-llms/10-rag-system-worked-lab.md)
- [Advanced RAG retrieval pattern](08-nlp-llms/11-advanced-rag-retrieval-patterns.md)
- [Reinforcement-learning algorithm lab](09-reinforcement-learning/04-algorithm-worked-lab.md)
- [Agent evaluation harness lab](10-agentic-ai/08-agent-evaluation-harness-lab.md)
- [Agent context, memory và persistence](10-agentic-ai/09-context-engineering-memory-and-persistence.md)
- [Agentic pattern và human control](10-agentic-ai/10-agentic-patterns-and-human-control.md)
- [Production model runbook](11-ai-engineering/10-production-model-runbook.md)
- [LLMOps evaluation và experiment design](11-ai-engineering/11-llmops-evaluation-and-experiment-design.md)
- [End-to-end Core AI Engineer Capstone](../projects/CORE_AI_ENGINEER_CAPSTONE.md)
