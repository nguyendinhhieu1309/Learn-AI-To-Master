# k-Nearest Neighbors and k-Means

The names are similar, but the tasks are different. **k-nearest neighbors (k-NN)** predicts a target using labeled nearby examples. **k-means** groups unlabeled examples around learned centroids.

## Learning objectives

- Explain training and prediction for k-NN and k-means.
- Calculate a small distance-based prediction by hand.
- State why feature scaling and distance choice matter.
- Derive the k-means objective and its alternating updates.
- Recognize when these methods fit the geometry poorly.

## 1. Distance is a modeling choice

For vectors $x,z\in\mathbb{R}^d$, Euclidean distance is

$$
d_2(x,z)=\sqrt{\sum_{j=1}^{d}(x_j-z_j)^2}.
$$

Manhattan distance sums absolute coordinate differences; cosine distance focuses on angle. A distance defines which cases count as “similar.” If age spans 0–100 while annual income spans 0–1,000,000, Euclidean distance may mostly reflect income. Fit feature scaling on training data and apply it unchanged at evaluation and inference.

## 2. k-nearest neighbors (k-NN)

To predict a new example:

1. calculate its distance to each training example;
2. select the nearest $k$ labeled examples;
3. for classification, take a majority vote (or distance-weighted vote);
4. for regression, average their targets (possibly with distance weights).

### Worked classification example

Suppose one-dimensional training points are $0\mapsto A$, $3\mapsto B$, and $4\mapsto B$. For query $x=1.2$, distances are 1.2, 1.8, and 2.8. With $k=1$, prediction is $A$; with $k=3$, the majority is $B$. Increasing $k$ smooths local decisions but can wash out small patterns. The value of $k$ is a hyperparameter selected using validation data.

A small $k$ has low bias and high variance; a large $k$ usually has higher bias and lower variance. In high-dimensional spaces, “nearest” examples may not be meaningfully close, and prediction can be expensive because many distances must be computed. Approximate-nearest-neighbor indexes can speed up large-scale retrieval but may trade exactness for latency.

## 3. k-means clustering

Given $n$ unlabeled points $x_i$ and a chosen cluster count $K$, k-means learns centroid $\mu_k$ and assignment $c_i$. It minimizes within-cluster squared Euclidean distance:

$$
J(c,\mu)=\sum_{i=1}^{n}\|x_i-\mu_{c_i}\|_2^2.
$$

The algorithm alternates two simple steps:

1. **Assign:** send each point to its nearest centroid.
2. **Update:** replace each centroid with the mean of its assigned points.

Each step cannot increase the objective for fixed other values, so the objective decreases or stays the same until assignments stop changing. The overall objective is non-convex; different starting centroids can produce different local solutions.

### Worked update on four points

Use one-dimensional points $0,2,8,10$ and $K=2$. Initialize centroids at $0$ and $10$. The assignments are $\{0,2\}$ and $\{8,10\}$. Their means are 1 and 9. Assigning again leaves the groups unchanged. The sum of squared distances is

$$
(0-1)^2+(2-1)^2+(8-9)^2+(10-9)^2=4.
$$

This describes compactness under the chosen feature representation; it does not prove the data truly contain two natural categories.

## 4. Select $k$ and evaluate clustering carefully

For k-NN, tune $k$, distance, weights, and preprocessing within training/validation splits. For k-means, inertia always decreases as $K$ grows, so the lowest inertia alone does not pick a useful number of clusters. Compare an elbow plot, silhouette score, stability under resampling, downstream utility, and whether clusters can be understood by domain experts. If labels exist, compare against an external metric, but do not pretend an unsupervised objective is the same as classification accuracy.

## 5. Limitations and alternatives

- k-NN can be slow at prediction time and sensitive to irrelevant dimensions, scale, and duplicated examples.
- k-means favors roughly spherical, similar-scale clusters under Euclidean distance. It struggles with elongated shapes, very different densities, and many outliers.
- A centroid is a mean and can be pulled by extreme values; k-medoids or robust methods may be preferable.
- Density-based clustering can find irregular shapes and mark noise; hierarchical clustering can expose a multiscale structure.

## 6. Python patterns

```python
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.neighbors import KNeighborsClassifier

knn = make_pipeline(StandardScaler(), KNeighborsClassifier(n_neighbors=5))
knn.fit(X_train, y_train)
```

```python
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler().fit(X_train)
X_scaled = scaler.transform(X_train)
clusters = KMeans(n_clusters=3, n_init="auto", random_state=0).fit(X_scaled)
```

Do not fit the scaler on future evaluation data. Use multiple k-means initializations because a single run can settle into a weak local solution.

## Practice with answer checks

1. If a feature is measured in dollars and another in years, what should you check before Euclidean-distance learning? **Answer:** scaling, feature relevance, and whether the distance represents meaningful similarity.
2. What are the new centroids for clusters $\{1,3,8\}$ and $\{10,12\}$? **Answer:** 4 and 11.
3. Does an inertia value prove that a cluster is a real-world group? **Answer:** no; it measures the chosen geometric objective only.
4. What does larger $k$ usually do in k-NN? **Answer:** smooths predictions, often reducing variance while increasing bias.

## Further reading

- [scikit-learn: KMeans](https://scikit-learn.org/stable/modules/generated/sklearn.cluster.KMeans.html)
- [scikit-learn: KNeighborsClassifier](https://scikit-learn.org/stable/modules/generated/sklearn.neighbors.KNeighborsClassifier.html)
