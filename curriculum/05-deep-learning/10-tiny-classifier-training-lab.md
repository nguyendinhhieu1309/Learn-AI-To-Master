# Tiny Classifier Training Lab: Shapes, Gradients, and Diagnostics

This lab trains a deliberately small classifier. The goal is to verify the mechanics of a deep-learning training run before scaling data, parameters, or hardware.

## Learning objectives

- Check batch and class dimensions before training.
- Use logits with a multiclass loss correctly.
- Separate training mode from evaluation mode.
- Demonstrate tiny-batch overfitting as a debugging test.
- Compare training and validation behavior without tuning on the test set.

## Prerequisites

Complete [neural networks](01-neural-networks-and-training.md), [backpropagation](05-backpropagation-worked-example.md), and [optimizers](09-optimizers-and-training-dynamics.md). Install a PyTorch build compatible with the learner's hardware using the official instructions; accelerator installation differs by platform.

## 1. Construct a tiny, separable dataset

The following points have two input features and two classes. This dataset is synthetic and exists only to inspect a training loop.

~~~python
import torch
from torch import nn

torch.manual_seed(7)

x_train = torch.tensor(
    [[-2.0, -1.0], [-1.0, -2.0], [-1.0, -1.0],
     [ 1.0,  1.0], [ 2.0,  1.0], [ 1.0,  2.0]],
    dtype=torch.float32,
)
y_train = torch.tensor([0, 0, 0, 1, 1, 1], dtype=torch.long)

x_valid = torch.tensor(
    [[-1.5, -0.5], [0.5, 1.5]],
    dtype=torch.float32,
)
y_valid = torch.tensor([0, 1], dtype=torch.long)

assert x_train.shape == (6, 2)
assert y_train.shape == (6,)
assert x_valid.shape == (2, 2)
~~~

The split is tiny and illustrative, not statistically reliable. For a real task, split by the true independent unit and preserve a final test set.

## 2. Define the model and loss

~~~python
model = nn.Sequential(
    nn.Linear(2, 8),
    nn.Tanh(),
    nn.Linear(8, 2),
)

loss_fn = nn.CrossEntropyLoss()
optimizer = torch.optim.AdamW(model.parameters(), lr=0.03, weight_decay=0.01)

logits = model(x_train)
assert logits.shape == (6, 2)
loss = loss_fn(logits, y_train)
print("initial loss:", float(loss.detach()))
~~~

The output has one logit per class. Pass raw logits to cross-entropy; do not apply softmax first. Labels are integer class indices with shape $(N,)$. A loss with a mismatched label shape can silently optimize a different problem or fail loudly, so inspect both shapes.

## 3. Train and validate

~~~python
from torch.nn.utils import clip_grad_norm_

history = []
for epoch in range(200):
    model.train()
    optimizer.zero_grad(set_to_none=True)
    train_logits = model(x_train)
    train_loss = loss_fn(train_logits, y_train)
    train_loss.backward()
    clip_grad_norm_(model.parameters(), max_norm=5.0)
    optimizer.step()

    model.eval()
    with torch.no_grad():
        valid_logits = model(x_valid)
        valid_loss = loss_fn(valid_logits, y_valid)
        valid_accuracy = (valid_logits.argmax(dim=1) == y_valid).float().mean()

    history.append(
        {
            "epoch": epoch,
            "train_loss": float(train_loss.detach()),
            "valid_loss": float(valid_loss),
            "valid_accuracy": float(valid_accuracy),
        }
    )
~~~

For a larger dataset, replace the full-batch tensors with a training data loader and accumulate metrics over all validation batches. `model.train()` and `model.eval()` change behavior for layers such as dropout and batch normalization. Evaluation should not update gradients.

## 4. Perform the tiny-batch overfit test

Train on only a few examples with enough model capacity and verify that training loss can become very small. This checks whether:

- inputs and labels are aligned;
- the loss is connected to model outputs;
- gradients reach trainable parameters;
- the optimizer actually updates them;
- the model can represent the task.

Overfitting a tiny batch is a debugging test, not evidence of generalization. If loss does not fall, inspect data types, labels, logits, gradient norms, learning rate, and whether parameters are frozen before adding layers.

## 5. Inspect predictions, not just one number

Print or plot:

- training and validation loss by epoch;
- training and validation accuracy;
- predicted class and probability for each example;
- gradient norm and whether clipping activated;
- model parameter count and inference batch size.

The validation set here has only two examples, so one error changes accuracy by 50 percentage points. Do not interpret the displayed score as a performance estimate.

## 6. Controlled variations

Change one factor at a time:

1. Reduce the learning rate by a factor of ten.
2. Increase hidden width.
3. Replace Tanh with ReLU.
4. Increase weight decay.
5. Remove clipping and inspect gradient norms.

For each run, record configuration, seed, train/validation metrics, and what changed. Keep the validation procedure fixed; do not choose settings based on a test set.

## Practice with answer checks

1. Why is the label tensor shape $(N,)$ rather than $(N,2)$ in this example?
2. Why should softmax not be applied before cross-entropy here?
3. What does successful tiny-batch overfitting establish?
4. What does it fail to establish?
5. Why is a two-example validation accuracy too noisy to compare models?

### Answers

1. The loss expects one integer class index for each example.
2. Cross-entropy expects logits and includes the log-softmax calculation internally; applying softmax first changes numerical behavior and the intended loss input.
3. It is evidence that the forward pass, labels, gradients, and optimizer can fit those examples.
4. It does not establish generalization, calibration, robustness, or useful performance on the target population.
5. Each example contributes 50 percentage points, so the score has extremely high sampling uncertainty.

## Common mistakes

- Feeding probabilities to a loss that expects logits.
- Evaluating while dropout or batch normalization remains in training mode.
- Reporting a validation score without the sample count.
- Increasing model capacity before checking the data and gradient path.
- Treating deterministic seeds as a guarantee of identical results on all hardware.

## References

- [PyTorch: Training with PyTorch](https://docs.pytorch.org/tutorials/beginner/introyt/trainingyt.html) — data loaders, loss functions, optimizers, and a complete training loop.
- [PyTorch: CrossEntropyLoss](https://docs.pytorch.org/docs/stable/generated/torch.nn.CrossEntropyLoss.html) — input and target conventions.
- [PyTorch: AdamW](https://docs.pytorch.org/docs/stable/generated/torch.optim.AdamW.html) — optimizer behavior and options.
