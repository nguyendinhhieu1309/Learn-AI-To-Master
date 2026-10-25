# Recommendation Systems

Recommendation systems rank items for users, such as products, videos, or documents. Their challenge is not only to predict preference: feedback is sparse, exposure is biased, and the best candidate depends on product constraints and user context.

## Learning objectives

- Distinguish explicit ratings from implicit interaction signals.
- Explain content-based, neighborhood, and latent-factor methods.
- Derive the matrix-factorization prediction and regularized objective.
- Select ranking metrics and a leakage-safe evaluation split.
- Recognize cold-start, exposure bias, and online experimentation issues.

## 1. What does the training data mean?

An explicit rating (for example, a 1–5 score) is a direct but often sparse preference signal. An implicit interaction (click, view, purchase, watch time) is abundant but ambiguous: no click may mean “not interested,” “never shown,” “already seen elsewhere,” or “did not notice.” Positive and negative examples must reflect how items were exposed.

Define the objective before fitting a model: predict a rating, rank relevant items, increase qualified engagement, or satisfy a constraint such as diversity. These objectives are not interchangeable.

## 2. Main model families

- **Popularity:** recommend globally or within a segment. It is a strong baseline and solves some cold-start cases.
- **Content-based:** represent users and items with metadata or text/vision features; rank items matching a user's observed interests. It can recommend new items with features but may over-specialize.
- **Neighborhood-based collaborative filtering:** recommend items liked by similar users or users similar to a target. Similarity can be computed from co-ratings or interactions.
- **Latent-factor models:** learn compact user and item vectors from interaction patterns.
- **Hybrid models:** combine collaborative signals, item features, context, and business constraints.

## 3. Matrix factorization

Let $R_{ui}$ be an observed rating from user $u$ for item $i$. Approximate the partially observed matrix with a user vector $p_u$ and item vector $q_i$:

$$
\hat R_{ui}=\mu+b_u+b_i+p_u^Tq_i,
$$

where $\mu$ is a global mean and $b_u,b_i$ are user/item bias terms. For observed rating pairs $\Omega$, fit the factors by minimizing

$$
J=\sum_{(u,i)\in\Omega}(R_{ui}-\hat R_{ui})^2
+\lambda\left(\sum_u\|p_u\|_2^2+\sum_i\|q_i\|_2^2\right).
$$

Only observed entries contribute to the rating loss; missing ratings are not automatically zeros. Regularization is needed because many factor pairs can explain sparse observations. Latent dimensions capture recurring patterns but are not guaranteed to have an intuitive meaning.

For implicit feedback, use a ranking loss or confidence-weighted objective rather than interpreting every absent interaction as a dislike. Pairwise objectives such as Bayesian Personalized Ranking train the score of an observed positive item to exceed a sampled negative item:

$$
\ell=-\log\sigma(s(u,i^+)-s(u,i^-)).
$$

The negative-sampling procedure materially affects what the model learns.

## 4. Offline evaluation

Use ranking metrics such as Recall@K, Precision@K, NDCG@K, MAP, or mean reciprocal rank when the product shows a ranked list. Each metric answers a different question: NDCG discounts lower positions; Recall@K measures how many relevant items appear in the first $K$; precision measures the fraction of shown items that are relevant under the chosen labels.

Split by time when the product recommends future interactions. For each user, ensure test interactions happen after training interactions; prevent future item metadata or user aggregates from leaking backward. Random row splits can place a user's future events in training and earlier events in test, producing an unrealistic estimate.

An offline test only measures performance on the logged exposure policy. A model may appear weak on items users were never shown, and a click is not automatically evidence of satisfaction. Online experiments need guardrails for latency, diversity, novelty, complaints, and unintended outcomes—not only clicks.

## 5. Cold start and production constraints

- New users have little interaction history; use onboarding signals, context, or safe popularity baselines.
- New items need content features or exploration to gather feedback.
- Sparse users/items produce uncertain factor estimates.
- Filtering rules, inventory, policy, and freshness constraints should be applied explicitly and logged.
- Candidate retrieval and ranking are often separate stages; retrieval must include good candidates for the ranker to select.

Track the exact exposure, recommendation, and feedback events. Without exposure logs, it is difficult to tell whether an item was ignored or never presented.

## Practice

1. Why is an unobserved user-item pair not necessarily a negative rating?
2. Which evaluation split is better for predicting next month's engagement: random rows or time-based? Why?
3. A new item has no interactions but good metadata. Which model family can rank it before collaborative signals arrive?
4. What is one reason an offline Recall@10 can disagree with an online result?

**Answer checks:** the item may never have been exposed; time-based, to match future prediction; content-based or hybrid; exposure bias, metric/product mismatch, feedback loops, or changed user behavior.
