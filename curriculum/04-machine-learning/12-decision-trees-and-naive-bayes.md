# Decision Trees and Naive Bayes

Decision trees build rules by splitting feature space. Naive Bayes compares class probabilities using Bayes' rule. They make different assumptions and often form useful, fast baselines before a more complex model is considered.

## Learning objectives

- Calculate entropy, Gini impurity, and information gain.
- Explain recursive splitting, stopping, and pruning.
- Distinguish a single tree, random forest, and gradient-boosted trees.
- Derive the Naive Bayes class score and understand smoothing.
- Recognize the assumptions and failure cases of both families.

## 1. A tree asks a sequence of questions

A decision tree asks questions such as “is account age below 30 days?” Each internal node applies a feature test; each leaf outputs a class distribution or numeric prediction. A path from root to leaf is an interpretable rule, though large trees can become difficult to understand.

## 2. Measure class impurity

Suppose a node contains class proportions $p_1,\ldots,p_K$. Entropy is

$$
H=-\sum_{k=1}^{K}p_k\log_2p_k,
$$

with $0\log 0$ defined as zero. Gini impurity is

$$
G=1-\sum_{k=1}^{K}p_k^2.
$$

Both are zero when every example has the same label and largest when classes are evenly mixed. For a candidate split into child nodes $c$, use a size-weighted impurity:

$$
I_{\mathrm{after}}=\sum_c\frac{n_c}{n}I(c).
$$

Information gain is parent entropy minus weighted child entropy. A tree chooses the feature/threshold with the largest gain (or greatest impurity reduction).

### Worked split example

At a node with four positive and four negative examples, entropy is $H(0.5,0.5)=1$ bit. A split creates one pure child of size four and another child containing two positive and two negative examples. Weighted child entropy is $\tfrac48(0)+\tfrac48(1)=0.5$, so information gain is $1-0.5=0.5$ bits. A split that makes both children pure would gain 1 bit.

## 3. Grow, stop, prune

Greedy trees repeatedly select the locally best split. They can overfit if grown too deeply, especially with small leaves. Control complexity with maximum depth, minimum leaf size, minimum impurity decrease, or cost-complexity pruning. Tune these using a validation split or cross-validation; select the final tree before evaluating on the test set.

Trees handle nonlinear interactions and mixed scales without requiring feature standardization. A single tree is unstable: small data changes may alter early splits. They can also favor some split opportunities and provide misleading importance rankings when features are correlated or have many possible split points.

## 4. Ensembles of trees

- **Bagging / random forest:** train many trees on bootstrap samples and random feature subsets, then average or vote. This mainly reduces variance and is robust on tabular data.
- **Gradient boosting:** add trees sequentially to reduce a loss, often following residual or gradient information. It can achieve strong accuracy but needs careful regularization and tuning.
- **Extremely randomized trees:** inject more split randomness, trading some bias for lower variance.

Ensembles are less directly interpretable than one small tree. Compare calibration, latency, memory, and error slices in addition to aggregate score. Feature importance is an association with model behavior, not a causal explanation.

## 5. Naive Bayes from Bayes' rule

For class $c$ and feature vector $x=(x_1,\ldots,x_d)$:

$$
P(c\mid x)\propto P(c)P(x\mid c).
$$

Naive Bayes assumes features are conditionally independent given the class:

$$
P(x\mid c)=\prod_{j=1}^{d}P(x_j\mid c).
$$

Therefore, compare classes by the log score

$$
\log P(c)+\sum_{j=1}^{d}\log P(x_j\mid c).
$$

The conditional-independence assumption is often wrong—for example, words in a sentence depend on one another—but classification can still work well, especially for sparse text features. Log space avoids multiplying tiny values.

### Multinomial Naive Bayes and smoothing

For word-count features, estimate each class's word probability from counts. Without smoothing, a word never observed in class $c$ gets probability zero, making the entire product zero. Additive smoothing with $\alpha>0$ gives

$$
P(\text{word }j\mid c)=\frac{N_{cj}+\alpha}{N_c+\alpha V},
$$

where $N_{cj}$ is the count for word $j$ in class $c$, $N_c$ is the total token count in that class, and $V$ is vocabulary size. The smoothing strength is a hyperparameter. Gaussian Naive Bayes instead models each continuous feature with a class-specific Gaussian; Bernoulli Naive Bayes models binary feature presence.

## 6. Compare models by task

For tabular data, compare a constant or linear baseline with a small tree and a regularized tree ensemble. For text counts, Naive Bayes can be very fast and competitive. Use the same data split, preprocessing rules, and metric for each candidate; inspect errors on important subgroups. A model's simplicity is valuable only if its performance and operational behavior fit the use case.

## Practice with answer checks

1. What is entropy for a pure two-class node? **Answer:** zero.
2. Why are child impurities weighted by child size? **Answer:** a split with a tiny pure child should not count as if it purified the whole node.
3. Why does Naive Bayes use log probabilities? **Answer:** to turn products into sums and avoid numerical underflow.
4. What does additive smoothing prevent? **Answer:** an unseen feature/class pair from forcing the entire class likelihood to zero.
5. Which tree ensemble mainly reduces variance by averaging randomized trees? **Answer:** random forest.

## Further reading

- [scikit-learn: DecisionTreeClassifier](https://scikit-learn.org/stable/modules/generated/sklearn.tree.DecisionTreeClassifier.html)
- [scikit-learn: Naive Bayes](https://scikit-learn.org/stable/modules/naive_bayes.html)
