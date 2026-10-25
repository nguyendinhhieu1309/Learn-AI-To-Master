# Neural Network Design, Losses, and Diagnostics

Architecture should reflect the structure of the input and the prediction task. This lesson connects design choices with the loss, output layer, and signs of training failure.

## 1. Computational graph and backpropagation

A network is a composition of differentiable operations. Autodiff records this computation and applies the chain rule backward to calculate gradients. For a layer $h=\sigma(Wx+b)$, gradients flow through the activation and the matrix multiplication to update both weights and earlier layers.

To debug a network, verify a tiny forward pass, output shape, label mapping, loss value, and gradient finiteness. Compare one gradient against finite differences on a tiny model when implementing custom operations.

## 2. Choose an output and loss that match the target

- Single-label multi-class: $C$ logits with cross-entropy.
- Binary classification: one logit with binary cross-entropy from logits.
- Multi-label classification: one independent sigmoid/logit per label; labels are not mutually exclusive.
- Regression: linear output with MSE, MAE, Huber, or a task-specific likelihood.
- Segmentation: per-pixel class logits with a pixel-wise loss, often combined with overlap-oriented terms.

Softmax is for mutually exclusive classes. Applying softmax to each output of a multi-label problem incorrectly forces probabilities to sum to one.

## 3. Activations and gradients

ReLU is simple and efficient but can produce inactive units; GELU and SiLU are smooth alternatives used in many architectures. Sigmoid can saturate for large magnitudes, making gradients small; it is common at a binary probability output but less common in deep hidden layers.

Initialization should account for fan-in/fan-out and activation. Residual connections help preserve information and gradient flow in deep networks. Diagnose vanishing or exploding gradients by logging gradient norms at several layers.

## 4. Normalization

Batch normalization uses batch statistics while training and running estimates at inference. Layer normalization normalizes within each example and is common in transformers. Group normalization can be useful for vision models with small batch sizes. These layers are not interchangeable in every architecture; validate train/eval behavior and distributed settings.

## 5. Architecture follows data structure

- MLPs: fixed-size tabular vectors or learned feature combinations.
- CNNs: local spatial patterns and translation-related structure in images.
- RNNs: sequential state with recurrent computation; useful in some streaming or compact settings.
- Transformers: attention over token or patch sequences, with position information supplied explicitly or through architecture.
- Graph neural networks: entities with relational edges.

Inductive bias can improve data efficiency. A transformer is not automatically best for small tabular data; compare simple baselines.

## 6. Diagnose learning curves

Training and validation loss should be interpreted together with task metrics. If both remain poor, investigate optimization, data, labels, and model capacity. If train improves while validation degrades, investigate overfitting or distribution mismatch. If loss improves but task metric does not, check class imbalance, thresholding, calibration, metric implementation, and label encoding.

## 7. Exercises

1. Why is softmax wrong for a task where each example can have several labels simultaneously?
2. Validation loss rises while training loss falls. What checks distinguish overfitting from a split bug?
3. A custom activation returns finite outputs but no gradients. What should be inspected?
4. Why might group normalization work better than batch normalization when GPU memory forces tiny batches?

## References

- [Dive into Deep Learning](https://d2l.ai/)
- [PyTorch: Loss functions](https://pytorch.org/docs/stable/nn.html#loss-functions)
