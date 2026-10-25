# Unsupervised Learning and Anomaly Detection

Unsupervised learning finds structure without target labels. Results are often descriptive rather than predictive, so validation requires careful task framing and domain review.

## 1. Clustering

K-means minimizes within-cluster squared distance to centroids. It is efficient, but requires choosing $k$, favors roughly spherical clusters, and is sensitive to scale and initialization. Run multiple initializations and inspect cluster stability.

Hierarchical clustering builds a merge tree and supports multiple granularities. DBSCAN groups dense regions and marks sparse points as noise; it can find non-spherical clusters but is sensitive to distance scale and varying densities. Gaussian mixture models estimate soft cluster membership under a mixture-distribution assumption.

## 2. Dimensionality reduction

PCA provides a linear projection maximizing retained variance. It is useful for compression, denoising, and visualization, but directions of high variance need not be predictive or semantically meaningful.

t-SNE and UMAP can help visualize local neighborhoods, but plot distances and cluster sizes should not be treated as faithful global geometry. Fit and tune visualizations for exploration; do not use a visually separated plot as sole evidence of true clusters.

## 3. Anomaly detection

An anomaly is defined relative to a process and consequence. Rare does not necessarily mean erroneous or dangerous. Isolation Forest isolates points through random partitions; One-Class SVM learns a boundary around typical examples; reconstruction methods flag inputs that reconstruct poorly. Each has assumptions and can fail on shifted or multimodal normal data.

When labeled anomalies exist, evaluate precision and recall at operational thresholds, alert volume, and time-to-detection. When labels are absent, conduct expert review, controlled synthetic tests, and monitor false-alarm burden. Avoid choosing a threshold from training scores alone.

## 4. Scaling and representation

Distance-based methods are sensitive to feature units and irrelevant dimensions. Standardize using training data. For high-dimensional data, select or learn representations with care and verify that results persist across seeds and samples.

## 5. Exercises

1. K-means separates clusters by income but ignores a categorical region code. What preprocessing and modeling questions arise?
2. A UMAP plot shows three islands. Why is that not enough to claim three natural populations?
3. An anomaly detector has 1% precision. What operational cost should you calculate before deployment?
4. A fraud team has no labels for the newest anomalies. What review loop could create evaluation data safely?

## References

- [scikit-learn: Clustering](https://scikit-learn.org/stable/modules/clustering.html)
- [scikit-learn: Novelty and outlier detection](https://scikit-learn.org/stable/modules/outlier_detection.html)
