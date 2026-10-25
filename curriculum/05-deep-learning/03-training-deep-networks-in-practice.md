# Training Deep Networks in Practice

This lesson turns the definitions of layers and losses into a disciplined training workflow. It applies to image, text, audio, and tabular neural networks.

## 1. A neural network as a parameterized function

A model maps input $x$ to prediction $f_\theta(x)$. Training minimizes an empirical objective over examples, often with regularization:

$$
J(\theta) = \frac{1}{N}\sum_{i=1}^{N} \ell\big(f_\theta(x_i), y_i\big) + \lambda\,\Omega(\theta)
$$

Here $\ell$ measures prediction error, $\Omega$ penalizes undesirable parameter settings, and $\lambda$ controls the tradeoff. For classification, pass logits to a numerically stable cross-entropy implementation; do not apply softmax twice.

## 2. The training loop

Each iteration performs a forward pass, loss calculation, gradient calculation, parameter update, and gradient reset. A minibatch estimates the full-dataset gradient with lower compute per update.

```python
for x, y in train_loader:
    optimizer.zero_grad(set_to_none=True)
    logits = model(x)
    loss = criterion(logits, y)
    loss.backward()
    torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
    optimizer.step()
```

Evaluate under inference mode, track validation metrics separately, and save the best checkpoint according to a metric chosen before training. Log optimizer, learning rate, batch size, seed, data version, model configuration, and checkpoint identity.

## 3. Optimizers and learning rate

Stochastic gradient descent updates parameters using a noisy minibatch gradient. Momentum accumulates a smoothed direction. Adam adapts each parameter’s step size using moving averages of gradients and squared gradients. AdamW decouples weight decay from Adam’s gradient update and is a common default for transformers.

The learning rate is often the most important optimization setting. Too high causes divergence or oscillation; too low makes progress slow or traps training in a poor region. Use warmup for some large-batch or transformer training, then decay with a schedule suited to the run. A short learning-rate range test can identify a safe order of magnitude.

If loss becomes NaN or Inf, check data values, logits, mixed precision, learning rate, and gradient norms. Gradient clipping can limit extreme updates but cannot repair incorrect labels or invalid operations.

## 4. Batch size and gradient accumulation

Larger batches improve hardware utilization but consume memory and produce fewer parameter updates per epoch. Gradient accumulation sums gradients across smaller microbatches to approximate a larger batch, while batch-normalization behavior remains tied to each microbatch. Keep the effective batch size and number of optimizer updates in experiment records.

## 5. Regularization and generalization

- Weight decay discourages overly large weights.
- Dropout randomly disables activations during training.
- Data augmentation creates plausible input variations.
- Early stopping selects a checkpoint before validation performance degrades.
- Label smoothing softens one-hot targets in some classification settings.

Regularization should match the problem. Strong augmentation can create unrealistic samples; dropout is not automatically useful everywhere. A validation gap may also come from data leakage, duplicate splitting, or a distribution shift.

## 6. Initialization, normalization, and activations

Poor initialization can make activations or gradients vanish or explode. Xavier-style initialization is commonly paired with tanh-like activations; He initialization is often used with ReLU-family activations. Normalization layers stabilize some architectures, but their train/eval modes matter. Batch normalization uses batch statistics during training and running statistics during inference; forgetting evaluation mode can change outputs.

Residual connections create shorter paths for gradient flow and support deeper networks. Monitor activation and gradient scales when training is unstable.

## 7. Mixed precision and reproducibility

Mixed precision can reduce memory and increase throughput on supported hardware. Use framework autocasting and gradient scaling when appropriate; numerically sensitive reductions may still need higher precision. Compare validation quality against a full-precision baseline.

Set seeds for repeatability, but do not promise bitwise identical runs across hardware, kernels, or distributed configurations. Report variability across several seeds when it affects the conclusion.

## 8. Debugging before scaling

Run these checks in order:

1. Verify tensor shapes, label ranges, class mapping, and preprocessing.
2. Overfit a tiny batch. If training loss cannot become very small, inspect the pipeline, loss, gradients, and optimizer.
3. Compare a simple baseline and a random-label sanity check.
4. Plot train/validation loss and task metrics each epoch.
5. Inspect actual predictions, not just aggregate scores.
6. Only then increase model size, data volume, or distributed complexity.

The tiny-batch test is a powerful implementation check, not evidence of generalization.

## 9. Compute and experiment tracking

Record the exact dataset snapshot and split IDs, source-code revision, dependencies, hardware, random seeds, training duration, and checkpoint. Track useful curves such as learning rate, gradient norm, throughput, GPU memory, loss, and validation metrics. A model artifact without its preprocessing and label mapping is incomplete.

## 10. Exercises

1. Training loss is flat from the first step. List four checks before changing the architecture.
2. Training accuracy rises while validation loss worsens. What should you inspect, and how do you choose the deployed checkpoint?
3. Mixed precision is twice as fast but occasionally produces NaN. What instrumentation and safeguards would you try?
4. Why might gradient accumulation not exactly match one physically large batch?

## References

- [PyTorch: Optimization tutorial](https://pytorch.org/tutorials/beginner/basics/optimization_tutorial.html)
- [PyTorch: AMP examples](https://pytorch.org/docs/stable/notes/amp_examples.html)
- [D2L: Optimization algorithms](https://d2l.ai/chapter_optimization/index.html)
