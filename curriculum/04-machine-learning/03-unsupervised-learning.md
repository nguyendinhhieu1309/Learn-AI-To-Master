# Unsupervised Learning

## Learning objectives

Understand what clustering and dimensionality reduction can reveal, and why results without labels require careful interpretation.

## 1. What does “unsupervised” mean?

Unsupervised methods look for structure in inputs without a target label supplied during fitting. They may group examples, create compact representations, or flag unusual points. A pattern is not automatically meaningful just because an algorithm returns it.

## 2. Clustering

**K-means** assigns points to `k` centers and iteratively updates assignments and centers to reduce within-cluster squared distance. It is simple and efficient, but assumes Euclidean distance is meaningful, needs `k`, and favors roughly spherical clusters. Scale numeric features first where units differ, fitting the scaler only on training data when evaluating a pipeline.

Hierarchical clustering creates nested groupings. Density-based methods such as DBSCAN can find irregular clusters and label some points as noise, but depend on distance and density settings.

## 3. Dimensionality reduction

PCA finds orthogonal directions capturing high variance and can compress or visualize data. High variance is not necessarily predictive or semantically important. Nonlinear visualization methods such as t-SNE or UMAP can make neighborhood patterns visible but may distort global distances. Treat a 2D plot as an exploration aid, not proof that there are exactly three natural groups.

## 4. Anomaly detection

Anomaly detection identifies points unlike a learned reference. Rare does not mean wrong or dangerous. Define what an anomaly means operationally and inspect examples. If labeled anomalies exist, evaluate with a suitable precision-recall analysis; if not, involve domain experts and test stability.

## 5. How to evaluate without ground truth

Check stability across seeds and small data changes, interpretability of clusters, downstream usefulness, and expert judgment. Internal measures such as silhouette score summarize geometric separation under a chosen distance; they do not prove business or scientific usefulness.

## Practice

Cluster a small dataset twice: once with raw features and once with scaled features. Compare assignments and explain which features dominate distance. Then inspect several examples per cluster and propose a descriptive label without claiming causation.

## Common mistakes

- Choosing a number of clusters only because it produces a visually pleasing chart.
- Applying distance-based methods to unscaled features with incompatible units.
- Interpreting PCA components or t-SNE axes as original variables.
- Treating an anomaly score as a verified error label.
