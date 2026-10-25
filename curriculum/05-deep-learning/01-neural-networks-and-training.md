# Neural Networks and Training

## Learning objectives

Explain a neural network as a composition of functions, follow a training step, and use learning curves to diagnose common problems.

## 1. One neuron

A neuron computes a weighted sum plus bias, then applies an activation:

$$
z=w^Tx+b,
\qquad a=\phi(z).
$$

The activation $\phi$ adds nonlinearity. Without nonlinear activations, stacking linear layers is still just one linear transformation. Common activations are

$$
\operatorname{ReLU}(z)=\max(0,z),
\qquad \sigma(z)=\frac{1}{1+e^{-z}}.
$$

## 2. Layers and shapes

For a batch `X` of shape `(batch_size, input_dim)`, a dense layer with `hidden_dim` units usually has a weight matrix shaped `(input_dim, hidden_dim)`, producing `(batch_size, hidden_dim)`. Track the batch axis and feature axis explicitly. A shape mismatch often indicates swapped axes or incorrect preprocessing.

A multi-layer network composes transformations:

$$
H^{(1)}=\phi(XW^{(1)}+b^{(1)}),
\qquad
H^{(2)}=\phi(H^{(1)}W^{(2)}+b^{(2)}),
$$

then computes output scores (logits). For multi-class classification, logits are converted to class probabilities with softmax.

## 3. Loss and backpropagation

The loss compares predictions with targets. For regression, mean squared error is common. For classification, cross-entropy is commonly applied to logits and integer class labels. Backpropagation computes how each parameter contributed to the loss using the chain rule; an optimizer uses those gradients to update parameters.

```python
for x_batch, y_batch in training_data:
    optimizer.zero_grad()
    logits = model(x_batch)
    loss = loss_function(logits, y_batch)
    loss.backward()
    optimizer.step()
```

Framework APIs differ, but the idea is consistent: clear old gradients, run a forward pass, compute loss, backpropagate, and update.

For a three-class task with four input features, a small PyTorch model could be:

```python
import torch.nn as nn

model = nn.Sequential(
    nn.Linear(4, 16),
    nn.ReLU(),
    nn.Linear(16, 3),  # raw class logits
)
loss_function = nn.CrossEntropyLoss()
```

`CrossEntropyLoss` expects raw logits and integer class IDs such as `0`, `1`, or `2`; do not apply softmax before this loss. At inference, use `model.eval()` and `with torch.no_grad():` so training-only behavior is disabled and gradients are not stored.

## 4. Batches, epochs, and validation

A **batch** is a subset used for one update. An **epoch** is one pass over the training set. Shuffling examples can help stochastic optimization when order has no meaning. Keep validation examples out of gradient updates. Use the validation curve to select settings or stop training; leave a test set for the final report.

## 5. Diagnose learning curves

- Training and validation loss both high: possible underfitting, poor features/labels, or optimization trouble.
- Training loss low but validation loss high: possible overfitting or train/validation mismatch.
- Loss becomes NaN or spikes: inspect learning rate, invalid inputs, normalization, and numerical stability.
- Metrics do not improve: verify target encoding, output layer, loss, and label alignment.

Dropout, weight decay, data augmentation, early stopping, and more representative data can help overfitting. More layers are not the first fix to try.

## Worked example

For a binary label, if a model assigns probability $p$ to the true class, its negative log-likelihood is

$$
\ell=-\ln p.
$$

For $p=0.8$, loss is $-\ln(0.8)\approx0.223$. If it assigns $p=0.1$, loss is $-\ln(0.1)\approx2.303$: confident wrong predictions are penalized more heavily.

## Practice

Train a small network on a toy classification task. Plot training and validation loss after each epoch. Change only one factor (for example, network width or weight decay) and describe what changed. Compare with a simple classical baseline.

## Common mistakes

- Applying softmax twice when the framework's cross-entropy loss expects logits.
- Updating weights using validation/test batches.
- Confusing batch size with the number of features.
- Trusting the final training score without inspecting held-out errors.
