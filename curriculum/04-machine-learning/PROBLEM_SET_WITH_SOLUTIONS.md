# Machine Learning Problem Set with Worked Solutions

This workbook checks whether you can choose and reason about common models, not just recall their names. Use the solutions after attempting each problem. Several questions intentionally ask for assumptions as well as arithmetic.

## Problems

### 1. Confusion-matrix metrics

A binary classifier produces TP = 18, FP = 6, FN = 12, TN = 64. Compute precision, recall, F1, and accuracy. Which metric would change if the decision threshold changed but ranking stayed fixed?

### 2. A misleading baseline

Only 5% of examples are positive. A classifier predicts every example as negative. Compute its accuracy and positive-class recall. Why is accuracy alone inadequate?

### 3. MAE versus RMSE

Residuals are $(1,1,8)$. Compute MAE and RMSE. Explain which is more affected by the large residual and why.

### 4. Linear-regression gradient step

Use $J(w,b)=\frac1{2m}\sum_i(wx_i+b-y_i)^2$ on $(0,1)$ and $(1,3)$, starting at $(0,0)$.

1. Find the gradients.
2. Update with learning rate 0.1.
3. Explain why feature scale can change the useful learning rate.

### 5. Logistic probability and loss

For logit $z=0$, compute $\sigma(z)$. For true label $y=1$, compute binary cross-entropy. What happens to the loss if the model instead predicts $p=0.01$?

### 6. Feature scaling and k-NN

Distance is Euclidean. A dataset has annual income in thousands (values near 80) and age in years (values near 40). Why might income dominate neighbor selection? Name a safe scaling procedure.

### 7. One k-means update

In one cluster, points are $(0,0)$, $(2,0)$, and $(4,3)$. Compute the centroid and explain why k-means is sensitive to initialization.

### 8. Entropy and information gain

A node contains three positive and one negative example. Compute its entropy in bits. A candidate split creates one pure positive child of size 2 and a child with one positive and one negative. Compute the weighted child entropy and information gain.

### 9. SVM margin

For a linear SVM with $w=(3,4)$, compute the geometric margin of the canonical separating hyperplanes. What changes if $w$ and the intercept are both multiplied by the same positive constant?

### 10. PCA interpretation

The eigenvalues of a centered covariance matrix are $(8,2,0)$. What fraction of variance is explained by the first principal component? Why does retaining 80% variance not guarantee retaining 80% of predictive information?

### 11. Choose a split

You have repeated measurements from 30 patients, 20 rows each, and want to predict outcomes for new patients. Why is a random row split misleading? What split unit should be used?

### 12. Leakage in preprocessing

You standardize each feature using the mean and standard deviation computed from all rows before cross-validation. Identify the problem and the corrected procedure.

### 13. Model comparison

A boosted-tree model scores 0.91 AUC and a linear model scores 0.89 AUC on one validation split. What evidence is still needed before claiming the tree model is preferable?

### 14. Cost-sensitive threshold

A calibrated risk model has false-positive cost 3 and false-negative cost 2. Under the two-action cost model, derive the probability threshold for action.

## Worked solutions

### Solution 1

$$
\text{precision}=\frac{18}{18+6}=0.75,\qquad
\text{recall}=\frac{18}{18+12}=0.60.
$$

$$
F_1=\frac{2PR}{P+R}=\frac{2(0.75)(0.60)}{1.35}\approx0.667,
\qquad
\text{accuracy}=\frac{18+64}{100}=0.82.
$$

Threshold-dependent metrics such as precision, recall, F1, and accuracy can change while ranking-based AUC remains unchanged.

### Solution 2

Accuracy is 95%; positive-class recall is 0. The classifier ignores every positive. Accuracy hides this because negatives dominate the population.

### Solution 3

$$
\operatorname{MAE}=\frac{1+1+8}{3}=\frac{10}{3}\approx3.33,
\qquad
\operatorname{RMSE}=\sqrt{\frac{1+1+64}{3}}=\sqrt{22}\approx4.69.
$$

Squaring makes the largest residual contribute disproportionately to RMSE.

### Solution 4

At the start, errors are $-1$ and $-3$. Therefore:

$$
\frac{\partial J}{\partial w}=\frac12((-1)0+(-3)1)=-1.5,\qquad
\frac{\partial J}{\partial b}=\frac12(-1-3)=-2.
$$

The updated parameters are $w=0.15$ and $b=0.2$. Large feature values multiply the slope gradient, so rescaling features changes the geometry and practical step size.

### Solution 5

$\sigma(0)=1/(1+e^0)=0.5$. The loss at $y=1$ is $-\log(0.5)\approx0.693$. At $p=0.01$, it is $-\log(0.01)\approx4.605$: confidently wrong predictions receive a large penalty.

### Solution 6

The numeric range and units of income can dominate Euclidean distance even if age is equally useful. Fit a standardizer using training data only, then apply that same transform to validation and test data. Scaling should reflect the problem; it is not automatically appropriate for every model.

### Solution 7

$$
\mu=\left(\frac{0+2+4}{3},\frac{0+0+3}{3}\right)=(2,1).
$$

Different initial centers can assign points to different clusters and lead to different local minima. Use multiple initializations and examine stability.

### Solution 8

The parent class probabilities are $(3/4,1/4)$:

$$
H_{\mathrm{parent}}=-\frac34\log_2\frac34-\frac14\log_2\frac14\approx0.811.
$$

The pure child has entropy 0. The balanced child has entropy 1. Each child has size 2, so weighted entropy is $0.5(0)+0.5(1)=0.5$. Information gain is about $0.811-0.5=0.311$ bits.

### Solution 9

For the canonical margin, distance from the separating hyperplane to either support hyperplane is $1/\|w\|_2$. Since $\|w\|_2=5$, the one-sided margin is 0.2 and the full margin width is 0.4. Multiplying both $w$ and intercept by a positive constant preserves the decision boundary but changes the canonical normalization; the max-margin objective fixes scale through its constraints.

### Solution 10

Total variance is $8+2+0=10$, so the first component explains $8/10=80\%$. PCA preserves variance, not target relevance. A low-variance direction can still separate classes or carry a rare but important signal.

### Solution 11

Rows from the same patient share patient-specific information. If rows for one patient occur in both train and test, the score can measure recognition of known patients rather than generalization to new ones. Group by patient and keep each patient in one partition.

### Solution 12

The held-out fold influenced fitted preprocessing statistics, so cross-validation no longer estimates the pipeline on unseen data. For each fold, fit the scaler on that fold's training partition and transform its validation partition with those fitted values.

### Solution 13

Check split uncertainty and repeated folds or seeds, the metric's confidence interval, calibration and task-specific operating points, error slices, latency, memory, cost, interpretability needs, and tuning-budget fairness. A 0.02 difference on one split is not conclusive by itself.

### Solution 14

Act when the expected false-positive cost is below the expected false-negative cost:

$$
(1-p)3 < p(2)
\quad\Longleftrightarrow\quad
p>\frac{3}{3+2}=0.6.
$$

This threshold assumes calibrated probabilities and that these costs capture the relevant decision tradeoff.

## Mastery check

You should be able to explain the assumptions behind each calculation, select a split that matches the claim, and defend a metric and threshold before comparing model families.
