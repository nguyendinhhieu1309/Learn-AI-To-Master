# Optimizers and Training Dynamics

An optimizer is a rule for changing parameters using gradients. It does not decide whether the labels are correct, whether the split is valid, or whether the model will generalize. Treat optimization as one part of the experiment, not as a substitute for one.

## Learning objectives

- Derive stochastic gradient descent and momentum updates.
- Explain Adam's first- and second-moment estimates and bias correction.
- Distinguish L2 regularization from decoupled weight decay in adaptive optimizers.
- Diagnose a learning rate that is too large or too small.
- Build a controlled training experiment and interpret its curves.

## Prerequisites

Know the chain rule and gradient descent. Complete [backpropagation](05-backpropagation-worked-example.md) and [training deep networks in practice](03-training-deep-networks-in-practice.md).

## 1. Full-batch and stochastic gradients

For dataset loss $J(\theta)=\frac1N\sum_{i=1}^{N}\ell_i(\theta)$, full-batch gradient descent uses every training example:

$$
\theta_{t+1}=\theta_t-\eta\nabla_\theta J(\theta_t).
$$

Stochastic gradient descent (SGD) estimates the gradient using one example or a mini-batch $B_t$:

$$
g_t=\frac1{|B_t|}\sum_{i\in B_t}\nabla_\theta\ell_i(\theta_t),
\qquad
\theta_{t+1}=\theta_t-\eta g_t.
$$

Mini-batches make updates cheaper and introduce sampling noise. That noise can be useful, but it also makes the loss curve fluctuate. Batch size changes memory use, throughput, gradient noise, and sometimes the best learning rate.

## 2. Momentum

Momentum keeps an exponentially weighted average of recent gradients:

$$
v_t=\beta v_{t-1}+g_t,\qquad
\theta_t=\theta_{t-1}-\eta v_t.
$$

Some implementations use $(1-\beta)g_t$ in the velocity definition; this rescales the state and requires a compatible learning-rate interpretation. Always identify the convention before comparing equations.

When gradients keep pointing in a similar direction, the velocity accumulates and moves faster. When gradients alternate, averaging can damp the oscillation. A large momentum coefficient also makes the optimizer slower to react when the useful direction changes.

## 3. Adam

Adam tracks an exponential average of the gradient and its elementwise square:

$$
m_t=\beta_1m_{t-1}+(1-\beta_1)g_t,
\qquad
v_t=\beta_2v_{t-1}+(1-\beta_2)g_t^2.
$$

Because both states start at zero, their early values are biased toward zero. Bias-corrected values are:

$$
\hat m_t=\frac{m_t}{1-\beta_1^t},
\qquad
\hat v_t=\frac{v_t}{1-\beta_2^t}.
$$

The update is:

$$
\theta_t=\theta_{t-1}-\eta
\frac{\hat m_t}{\sqrt{\hat v_t}+\epsilon}.
$$

The square root, division, and $\epsilon$ are applied elementwise. The second-moment estimate adapts the scale per parameter. Adam is not guaranteed to be best for every dataset or architecture; compare it with a meaningful baseline such as SGD with momentum.

## 4. One Adam step by hand

For a single parameter, let $\theta_0=1$, gradient $g_1=2$, $\beta_1=0.9$, $\beta_2=0.999$, $\eta=0.1$, and ignore the tiny $\epsilon$ for this arithmetic.

$$
m_1=0.1(2)=0.2,\quad
v_1=0.001(2^2)=0.004,
$$

$$
\hat m_1=\frac{0.2}{0.1}=2,\qquad
\hat v_1=\frac{0.004}{0.001}=4.
$$

Thus the step is $0.1(2/\sqrt4)=0.1$ and $\theta_1=0.9$. Bias correction matters: without it the first step would be much smaller.

## 5. Adam versus AdamW

Adding an L2 penalty to the objective changes the gradient:

$$
J_{\mathrm{reg}}(\theta)=J(\theta)+\frac{\lambda}{2}\|\theta\|_2^2,
\qquad
\nabla J_{\mathrm{reg}}=\nabla J+\lambda\theta.
$$

For plain SGD this resembles multiplying parameters by a shrinkage factor each step. With an adaptive optimizer, putting $\lambda\theta$ into the gradient also changes the optimizer's moment estimates. Decoupled weight decay instead applies shrinkage separately:

$$
\theta_t=\theta_{t-1}
-\eta\frac{\hat m_t}{\sqrt{\hat v_t}+\epsilon}
-\eta\lambda\theta_{t-1}.
$$

This is the core distinction in AdamW. The practical choice of which parameters to decay—often excluding biases or normalization parameters—is a modeling convention that should be stated and validated.

## 6. Learning-rate schedules

The learning rate controls the scale of the update. Common schedules include:

- a short warm-up when early updates are unstable at the target rate;
- step decay at predefined milestones;
- cosine decay over a fixed training budget;
- reducing the rate when validation performance stops improving.

A schedule is part of the training procedure. Record it, because “same model and optimizer” is not a reproducible comparison if the schedule differs.

## 7. Gradient clipping

If the global gradient norm exceeds threshold $c$, norm clipping rescales it:

$$
g\leftarrow g\cdot\min\left(1,\frac{c}{\|g\|_2}\right).
$$

Clipping limits unusually large updates; it does not fix a wrong loss, bad data, or an inappropriate learning rate. Log how often clipping activates. If it is active almost every step, diagnose the cause.

## 8. Read the curves before changing the model

| Observation | First checks |
| --- | --- |
| Training and validation loss remain high | labels, preprocessing, capacity, learning rate |
| Training loss falls; validation loss rises | overfitting, leakage-free split, regularization, data mismatch |
| Loss becomes NaN or jumps sharply | learning rate, non-finite inputs, loss/logit stability, gradient norm |
| Loss barely changes | frozen parameters, zero gradients, wrong target, too-small rate |
| Training improves but task metric does not | loss/metric mismatch, threshold, class balance, calibration |

Change one factor at a time and preserve a known-good run. Plot both loss and task metrics; a training objective is not always the deployment objective.

## 9. A controlled optimizer experiment

1. Fix the data split, initialization, model, batch size, and number of updates.
2. Try a small logarithmic sweep of learning rates.
3. Compare SGD with momentum and AdamW only after the pipeline can overfit a tiny batch.
4. Keep validation-based choices separate from final test reporting.
5. Record optimizer settings, seed, hardware, precision, and best validation checkpoint.
6. Repeat promising settings with multiple seeds if compute allows; report variability.

Do not compare only the best seed. A method that wins once but fails often may be less useful than a slightly weaker but stable one.

## Practice with answer checks

1. Why does Adam use bias correction?
2. In the one-step example, why is the bias-corrected update magnitude independent of the gradient magnitude for this single positive gradient?
3. Why is L2 gradient addition not identical to decoupled weight decay for Adam?
4. What should you inspect if training loss becomes NaN?
5. Why is learning rate usually the first optimizer parameter to tune?

### Answers

1. The moving averages begin at zero, so their early values underestimate the moments. Correction accounts for that initialization bias.
2. With one positive scalar gradient, the first corrected moments are $g$ and $g^2$, so their ratio is $g/|g|=1$. The update size becomes the learning rate; this does not mean the behavior is scale-invariant in later noisy multidimensional training.
3. Adding L2 changes the gradient passed through the moment estimates. Decoupled decay shrinks parameters separately from those estimates.
4. Check input and target finiteness, logits, loss computation, learning rate, gradient norms, and mixed-precision scaling before increasing model complexity.
5. It directly multiplies every update and can move training from no progress to instability. Other settings are hard to interpret before the step scale is reasonable.

## Common mistakes

- Assuming Adam removes the need to tune a learning rate.
- Confusing a training-loss minimum with the best validation checkpoint.
- Applying gradient clipping without monitoring how often it activates.
- Comparing optimizers with different update budgets or batch sizes.
- Describing AdamW as “Adam plus L2” without explaining the decoupling.

## References

- [PyTorch: AdamW](https://docs.pytorch.org/docs/stable/generated/torch.optim.AdamW.html) — algorithm, parameters, and implementation notes.
- [PyTorch: Optimizers](https://docs.pytorch.org/docs/stable/optim.html) — optimizer state, update steps, and available methods.
