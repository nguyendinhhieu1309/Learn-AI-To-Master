# Mathematics for AI

## Outcomes

- Read vector and matrix expressions used in machine learning.
- Relate derivatives and gradients to model optimization.
- Use probability and statistics to reason about uncertainty and evidence.

## Prerequisites

Basic arithmetic and algebra are enough to begin. Review missing topics as they appear; study this module alongside Python.

## Suggested sequence

1. [Algebra, functions, and notation](00-algebra-functions-notation.md)
2. [Vectors and matrices](01-vectors-and-matrices.md)
3. [Calculus and optimization](02-calculus-and-optimization.md)
4. [Probability and statistics](03-probability-and-statistics.md)
5. [Statistical inference and experiment design](04-statistical-inference-and-experiment-design.md)
6. [Linear algebra for machine learning](05-linear-algebra-for-machine-learning.md)
7. [Information theory and numerical computation](06-information-theory-and-numerical-computation.md)
8. [Probability distributions and Bayes](07-probability-distributions-and-bayes.md)
9. [Likelihood, MLE, and MAP](08-likelihood-mle-and-map.md)
10. [Convex optimization and constraints](09-convex-optimization-and-constraints.md)

After the lessons, use the [problem set with worked solutions](PROBLEM_SET_WITH_SOLUTIONS.md) to check whether you can derive and calculate the core results without copying an example.

### How to study each lesson

For every displayed equation, first identify the input, output, parameter, and shape of each symbol. Then work the numeric example by hand, explain the result in plain language, and only after that translate it into NumPy or PyTorch. Keep a notebook of formulas you can derive and assumptions you need to check; memorizing a formula without knowing its dimensions or assumptions is not enough.

You do not need to finish every advanced topic before beginning ML. Complete lessons 00–03 first; then study lessons 04–09 alongside model evaluation and the algorithms that use them.

## Practice

- Compute a dot product and describe what changes when a vector is scaled.
- Differentiate a simple quadratic function and use gradient descent to approach its minimum.
- Simulate coin flips, estimate a probability, and observe how the estimate changes with sample size.
- Compare mean and median on a dataset containing an outlier.
- Derive the gradient of squared error for a one-feature linear model and verify one component with a finite difference.
- Compute a Bernoulli maximum-likelihood estimate from coin-flip counts, then explain how a Beta prior changes the estimate.

## Checkpoint project

Implement linear regression for a tiny dataset using a closed-form solution and gradient descent. Plot the loss as training proceeds, compare the answers, and explain how feature scale affects optimization.

## Common mistakes to watch for

- Treating a matrix as an untyped list of numbers: track dimensions and what each axis represents.
- Confusing correlation with causation.
- Reading a derivative as a model score instead of a local rate of change.
- Applying a formula without checking its assumptions.
- Mixing a probability such as $P(A\mid B)$ with the reversed conditional $P(B\mid A)$.
- Losing a transpose or the batch axis when turning a scalar equation into a matrix equation.
- Treating a derivative as the exact change for a large step instead of a local approximation.
