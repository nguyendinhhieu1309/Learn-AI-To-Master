# AI Learning Cheat Sheet

A quick reference for concepts, equations, metrics, and workflows in this repository. Use it to recall material after studying the full lessons; this sheet is not a replacement for worked examples and practice.

## 1. A reliable ML workflow

1. **Define the task:** what is known at prediction time, what output is needed, who uses it, and what mistakes cost.
2. **Inspect the data:** row meaning, schema, units, missingness, labels, duplicates, collection bias, and license.
3. **Split before learning transforms:** choose random, group, or time-aware splits to reflect deployment.
4. **Create a simple baseline:** majority class, mean/median, domain rule, lexical search, or small model.
5. **Build a pipeline:** fit imputation, scaling, feature selection, and model only on training folds.
6. **Evaluate with suitable metrics:** use validation for choices and keep the test set for a final estimate.
7. **Inspect errors and subgroups:** find patterns, not just one aggregate score.
8. **Record and reproduce:** data version, code, settings, dependencies, model, and preprocessing.
9. **Deploy with checks:** validate inputs, measure latency/cost, monitor quality, and provide rollback.

## 2. Math at a glance

### Linear algebra

Key vector operations:

$$
\begin{aligned}
\mathbf{w}^{\top}\mathbf{x} &= \sum_{j=1}^{d} w_jx_j, \\
\lVert\mathbf{x}\rVert_2 &= \sqrt{\sum_{j=1}^{d}x_j^2}, \\
\operatorname{cos}(\mathbf{u},\mathbf{v}) &=
\frac{\mathbf{u}^{\top}\mathbf{v}}
{\lVert\mathbf{u}\rVert_2\lVert\mathbf{v}\rVert_2}.
\end{aligned}
$$

The dot product combines aligned components; the Euclidean norm measures vector length; cosine similarity compares direction.

For matrix shapes, if $X\in\mathbb{R}^{n\times d}$ and $W\in\mathbb{R}^{d\times k}$, then $XW\in\mathbb{R}^{n\times k}$. A linear prediction is $\hat{y}=\mathbf{w}^{\top}\mathbf{x}+b$; for a batch, $\hat{\mathbf{y}}=X\mathbf{w}+b$. Rows often represent examples and columns features, but verify each library's convention.

### Calculus and optimization

A derivative is a local rate of change. A gradient is the vector of partial derivatives and points toward the direction of greatest local increase.

Gradient descent:

$$
\theta \leftarrow \theta-\eta\nabla_{\theta}J(\theta),
$$

where $\eta$ is the learning rate.

Regularized objective:

$$
J_{\mathrm{total}}=J_{\mathrm{data}}+\lambda\Omega(\theta).
$$

Chain rule: for $f(g(x))$, the derivative is $f'(g(x))g'(x)$.

If loss diverges, first inspect the learning rate, input scale, invalid values, and gradient magnitudes. If training improves but validation worsens, investigate overfitting or a split mismatch.

### Probability and statistics

$$
\begin{aligned}
P(\neg A) &= 1-P(A), \\
P(A\mid B) &= \frac{P(A\cap B)}{P(B)}, \\
P(A\mid B) &= \frac{P(B\mid A)P(A)}{P(B)}, \\
\bar{x} &= \frac{1}{n}\sum_{i=1}^{n}x_i, \\
\operatorname{Var}(X) &= \mathbb{E}\left[(X-\mathbb{E}[X])^2\right].
\end{aligned}
$$

The third equation is Bayes' rule; standard deviation is the square root of variance. A sample may be biased even if it is large. Correlation alone does not establish cause.

## 3. Data handling reminders

- Missing values have meanings; inspect why they are missing before replacing them.
- Fit imputers, scalers, encoders, feature selection, and PCA on training data only.
- Random split: independent examples. Group split: related examples must stay together. Time split: train on past, evaluate on future.
- Leakage question: “Would this feature be available at the real prediction time?”
- Keep raw data unchanged; document cleaning decisions and preserve source/license information.
- For pandas, useful first checks include **df.shape**, **df.dtypes**, **df.head()**, **df.isna().sum()**, and **df.nunique()**.
- Before a SQL join, check key uniqueness and row counts. **WHERE** filters rows; **GROUP BY** aggregates; **HAVING** filters groups; use **IS NULL** for missing SQL values.
- CSV is portable, Parquet is typed and columnar, and JSON supports nested records; choose based on the workload and preserve raw-source provenance.
- For streaming/time series, distinguish event time from arrival time and compute features using only information available at the prediction cutoff.
- For a join, state each table's row grain and expected key cardinality; check row counts and aggregate totals afterward.
- For scheduled pipelines, prefer deterministic/idempotent steps, explicit watermarks, quality gates, lineage, and versioned outputs.
- Offline training and online serving must use point-in-time-correct, semantically identical feature definitions.

## 4. Software engineering essentials

- Separate data ingestion, feature logic, training, inference, API, and configuration into clear modules.
- Use environments, dependency records, Git diffs, structured logs, typed interfaces, and focused error handling.
- Test layers: unit → schema/contract → integration → end-to-end. Model evaluations are statistical checks alongside software tests.
- Define input/output fields, types, units, missing behavior, label meaning, timeout, and error responses for every model API.
- Retries need caps/backoff; side-effecting actions need idempotency keys and authorization.

## 5. Python and data structures

| Structure or algorithm | Typical complexity | Reminder |
| --- | --- | --- |
| List indexing | $O(1)$ | Access by position |
| List search | $O(n)$ | Linear scan |
| List append | amortized $O(1)$ | Add to the end |
| Insert/remove at list front | $O(n)$ | Later elements must shift |
| Dict/set lookup, insert, delete | average $O(1)$; worst $O(n)$ | Hash table |
| Deque operations at either end | $O(1)$ | Useful for a queue |
| Heap minimum lookup | $O(1)$ | Minimum is at the root |
| Heap push/pop | $O(\log n)$ | Useful for priorities |
| Linked-list search | $O(n)$ | No random indexing |
| Linked-list head insertion | $O(1)$ | When the head/node is known |
| Balanced BST search/insert/delete | $O(\log n)$ | An unbalanced BST can degrade to $O(n)$ |
| Graph adjacency-list storage | $O(V+E)$ | BFS/DFS also take $O(V+E)$ |
| Linear search | $O(n)$ | Does not require sorting |
| Binary search | $O(\log n)$ | Requires sorted random-access input |
| General comparison sort | typically $O(n\log n)$ | Quadratic sorts mainly suit learning or tiny inputs |

Time and space complexity describe growth with input size, not exact seconds on a particular machine. Follow the [Python, OOP, and DSA learning sequence](curriculum/02-python/README.md).

## 6. Machine learning

### Common tasks

- **Regression:** predict a number.
- **Classification:** predict a category or category probabilities.
- **Clustering:** group examples without labels; groups need interpretation and validation.
- **Dimensionality reduction:** create a lower-dimensional representation; information or interpretability may be lost.

### Losses and metrics

$$
\begin{aligned}
\operatorname{MSE} &= \frac{1}{n}\sum_{i=1}^{n}(y_i-\hat{y}_i)^2, \\
\operatorname{MAE} &= \frac{1}{n}\sum_{i=1}^{n}|y_i-\hat{y}_i|, \\
\operatorname{Precision} &= \frac{TP}{TP+FP}, \\
\operatorname{Recall} &= \frac{TP}{TP+FN}, \\
F_1 &= \frac{2PR}{P+R}, \\
\operatorname{Accuracy} &= \frac{TP+TN}{TP+TN+FP+FN}, \\
\operatorname{IoU} &= \frac{\text{intersection area}}{\text{union area}}.
\end{aligned}
$$

Precision asks how many predicted positives were correct; recall asks how many actual positives were found. F1 balances precision and recall. Accuracy can mislead with class imbalance. IoU is used for boxes and masks.

**TP** = true positive · **FP** = false positive · **FN** = false negative · **TN** = true negative.

Choose the metric and classification threshold based on error costs. Report counts and class-level errors as well as aggregate metrics. Avoid tuning repeatedly on the final test set.

### Model intuition

- Linear/logistic models: simple, fast baselines; coefficients need careful interpretation.
- Decision trees: rule-like nonlinear splits; deep trees can overfit.
- Random forests: average many randomized trees.
- Gradient boosting: add weak learners sequentially to correct previous errors.
- k-means: assign points to $k$ centers; scale features and inspect assumptions.
- PCA: directions of high variance; useful for compression, not automatically predictive.

## 7. Deep learning

Neuron:

$$
z=\mathbf{w}^{\top}\mathbf{x}+b,\qquad a=\phi(z).
$$

Softmax and cross-entropy:

$$
p_i=\frac{\exp(z_i)}{\sum_j\exp(z_j)},\qquad
\mathcal{L}=-\sum_i y_i\log(p_i).
$$

Cross-entropy strongly penalizes assigning low probability to the correct class. A training step is forward pass → loss → backpropagate gradients → optimizer update. A batch is the number of examples per update; an epoch is one pass through the training data.

- CNNs use shared local filters to capture spatial patterns.
- Transformer attention mixes information based on learned similarity:

$$
\operatorname{Attention}(Q,K,V)=
\operatorname{softmax}\left(\frac{QK^{\top}}{\sqrt{d_k}}\right)V.
$$

- Always compare with a baseline and monitor validation behavior, not only training loss.

## 8. Computer vision

- An image array often has shape $(H,W,C)$ for height, width, and channels. Pixel indexing often uses $[\text{row},\text{column}]=[y,x]$.
- Check channel order (RGB vs. BGR), value range, resize, crop, and normalization.
- **Classification:** one or more labels for an image.
- **Detection:** class plus bounding box; IoU measures overlap; AP summarizes a precision-recall curve.
- **Semantic segmentation:** class per pixel; **instance segmentation** separates individual objects.
- Transform labels when transforming images. Keep nearby frames or same-scene images in one split when leakage is possible.

## 9. 3D vision and robotics

Name every coordinate frame and unit. The transform $T_{A\leftarrow B}$ converts coordinates from frame B into frame A. A homogeneous transform is:

$$
T_{A\leftarrow B}=
\begin{bmatrix}
R_{A\leftarrow B} & \mathbf{t}_{A\leftarrow B} \\
\mathbf{0}^{\top} & 1
\end{bmatrix}.
$$

Composition order matters. Pinhole projection and depth back-projection:

$$
\begin{aligned}
u &= f_x\frac{X}{Z}+c_x, &
v &= f_y\frac{Y}{Z}+c_y, \\
X &= \frac{(u-c_x)Z}{f_x}, &
Y &= \frac{(v-c_y)Z}{f_y}.
\end{aligned}
$$

RGB-D/point clouds may contain invalid depth, motion, and calibration errors. Preserve timestamps and frame IDs. Perception output is not a safe motion command; check confidence, workspace, constraints, and latency.

## 10. NLP and LLM systems

- Tokens are model input units; token IDs are not semantic distances.
- A transformer uses attention and feed-forward layers; a causal language model predicts the next token.
- Generation controls such as temperature and top-p change sampling behavior, not factual reliability.
- RAG pipeline: prepare/index documents → retrieve → optionally rerank → answer from evidence → validate citations.
- Evaluate retrieval and answer quality separately. Include unanswerable questions and measure appropriate abstention.
- Retrieved text and user input are untrusted. Enforce access controls in code, not only in prompts.
- Prompting changes instructions; RAG supplies current evidence; fine-tuning adapts behavior/format. Choose based on the measured failure source.
- Evaluate extraction, retrieval, generation, tool execution, privacy, latency, and cost as parts of one user task.

## 11. AI systems engineering

- Batch jobs favor throughput; online APIs favor latency/availability; async jobs suit long-running tasks.
- Measure end-to-end p50/p95/p99 latency, queue depth, errors, model quality, and cost per successful task.
- Logs record events, metrics show aggregate behavior, and traces connect one request across components.
- SLI = measured service property; SLO = target over a time window. Attach alerts to owners and runbooks.
- Use containers, CI/CD, cloud IAM, rollback, bounded queues/retries, and minimum permissions.

## 12. Reinforcement learning

An MDP contains states $\mathcal{S}$, actions $\mathcal{A}$, transition dynamics, reward $R$, and discount factor $\gamma$. Discounted return:

$$
G_t=r_{t+1}+\gamma r_{t+2}+\gamma^2r_{t+3}+\cdots.
$$

- $V(s)$ is expected future return from state $s$.
- $Q(s,a)$ is expected return after action $a$ in state $s$.
- Epsilon-greedy explores randomly with probability $\epsilon$; otherwise it selects a best-known action.
- Evaluate across multiple seeds, compare baselines, inspect trajectories, and test reward loopholes.

Q-learning update:

$$
Q(s,a)\leftarrow Q(s,a)+\alpha\left[
r+\gamma\max_{a'}Q(s',a')-Q(s,a)
\right].
$$

## 13. Agentic AI

- Prefer a fixed workflow when the steps are known; use an agent when model-directed choice adds measurable value.
- Define task boundary, allowed tools, typed inputs/outputs, maximum steps, timeout, cost budget, and stop rule.
- Validate tool arguments and results in program code. Use least privilege, idempotency, and clear errors.
- Treat tool output and retrieved documents as untrusted input; prompt injection is a system-security concern.
- Evaluate task success, tool correctness, invalid actions, recovery, latency, cost, and safety failures.
- Do not claim completion without evidence that tools succeeded.
- **Skill** = reusable procedure/instructions; **tool** = executable capability; **memory/state** = retained information; **workflow** = control flow; **harness** = runtime/context/policies around the agent.
- Agent Skills commonly package a SKILL.md file with optional references/assets/scripts. Review scripts and permissions; a skill is not a sandbox.
- LangChain create_agent is a higher-level model/tool loop; LangGraph exposes explicit graph state, nodes, routing, checkpoints, and pause/resume.
- MCP standardizes discovery/calling of tools and access to resources/prompts; it does not authorize access or guarantee safety.
- Multi-agent patterns include router/handoff, manager-as-tool, sequential pipeline, and parallel fan-out/fan-in. Compare each with a single-agent baseline.

## 14. MLOps quick checklist

- Version data, code, dependencies, preprocessing, model, and configuration.
- Validate the inference input schema and keep training/inference transforms aligned.
- Measure latency percentiles, throughput, memory, errors, and data quality.
- Drift is a signal to investigate, not proof of model failure; stable inputs do not prove good performance.
- Define alert owner, response, rollback path, privacy, retention, and maintenance responsibility.

## 15. Responsible AI reminders

- Define who benefits, who may be harmed, and where the system is not appropriate.
- Measure relevant subgroups with sample sizes and uncertainty; overall metrics can hide uneven errors.
- Fairness criteria express different goals and can conflict; state the chosen trade-offs.
- Minimize data, check permission/retention, and protect logs and artifacts.
- Human oversight needs authority and a usable appeal/correction path.
- Reassess risks when data, model, interface, or deployment context changes.

## 16. Debugging prompts

When a result looks wrong, ask in this order:

1. Are the labels aligned with the examples?
2. Are the shapes, types, units, and coordinate frames correct?
3. Is there leakage or a split that differs from deployment?
4. Does a simple baseline work?
5. Are preprocessing and inference identical?
6. Which examples fail, and are those labels/data trustworthy?
7. Can another person reproduce the run and the metric?

## Related lessons

Use the [curriculum map](curriculum/README.md) to open the detailed explanations and exercises for each topic.
