# Information Theory and Numerical Computation

Information theory explains uncertainty and coding; numerical analysis explains why mathematically equivalent formulas can behave differently on computers.

## 1. Entropy and cross-entropy

For a discrete random variable with probabilities $p_i$, entropy is:

$$
H(P)=-\sum_i p_i\log p_i
$$

Entropy is high when probability mass is spread across many outcomes and low when one outcome dominates. The logarithm base changes units: base 2 gives bits; natural log gives nats.

Cross-entropy between target distribution $P$ and predicted distribution $Q$ is:

$$
H(P,Q)=-\sum_i p_i\log q_i
$$

For one-hot labels, minimizing cross-entropy rewards high probability on the correct class. KL divergence measures the extra coding cost from using $Q$ instead of $P$:

$$
D_{\mathrm{KL}}(P\|Q)=\sum_i p_i\log\frac{p_i}{q_i}
=H(P,Q)-H(P)
$$

It is nonnegative but asymmetric, so it is not an ordinary distance.

## 2. Log probabilities and stable softmax

Directly computing $e^{z_i}$ can overflow for large logits. Subtracting the maximum logit leaves softmax unchanged and improves stability:

$$
\operatorname{softmax}(z_i)=
\frac{e^{z_i-m}}{\sum_j e^{z_j-m}},\qquad m=\max_j z_j
$$

Use a framework’s stable `log_softmax` or cross-entropy-from-logits implementation instead of manually taking `log(softmax(...))`. For sums of probabilities, log-sum-exp provides a stable calculation.

## 3. Floating-point limitations

Computers store approximate floating-point numbers. Small rounding errors accumulate; adding a very small value to a very large one may lose the smaller value. Algebraic rearrangements can produce different numerical errors. Avoid exact equality checks for computed floats; use a tolerance tied to the scale and purpose.

Other common issues include overflow, underflow, division by nearly zero, ill-conditioned matrices, and non-finite inputs. Monitor loss and gradient finiteness. Normalize input scales where appropriate and prefer numerically stable library routines over hand-coded inverses.

## 4. Why not compute a matrix inverse?

To solve $Ax=b$, use a linear solver rather than explicitly forming $A^{-1}$ and multiplying. The solver is typically faster, more stable, and can exploit matrix structure. If $A$ is singular or poorly conditioned, inspect the model/data and consider regularization or a different formulation.

## 5. Entropy in ML practice

Entropy appears in decision-tree splits, uncertainty estimates, language modeling, and regularized policy learning. Predictive entropy alone does not tell whether a model is wrong: a confidently wrong model has low entropy. Compare uncertainty methods against actual error and calibration on held-out data.

## 6. Exercises

1. What is the entropy of a deterministic binary outcome? What about a fair coin, in bits?
2. Why is cross-entropy used with class probabilities rather than squared error for ordinary classification?
3. Why can `softmax([1000, 1001])` overflow in a naive implementation, and how does subtracting the maximum help?
4. Why should solving $Ax=b$ generally avoid explicit matrix inversion?

## References

- [Deep Learning: Information Theory](https://www.deeplearningbook.org/contents/infotheory.html)
- [NumPy: Linear algebra](https://numpy.org/doc/stable/reference/routines.linalg.html)
