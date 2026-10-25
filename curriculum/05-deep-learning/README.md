# Deep Learning

## Outcomes

- Derive a small network's forward pass, loss, and backpropagated gradients.
- Track activations, parameters, and masks by batch, token, channel, and spatial shape.
- Explain the inductive biases and computational costs of MLPs, CNNs, transformers, and generative models.
- Train a model with a valid split, stable optimization, reproducible records, and useful error analysis.

## Prerequisites

Python, arrays/tensors, basic probability, linear algebra, derivatives, and the ML evaluation workflow.

## Suggested sequence

1. [Neural networks and training](01-neural-networks-and-training.md)
2. [Backpropagation worked example](05-backpropagation-worked-example.md)
3. [Convolution and CNN shapes](06-convolution-and-cnn-shapes.md)
4. [Attention and transformers by hand](07-attention-and-transformers-by-hand.md)
5. [CNNs, transformers, and generative models overview](02-cnn-transformers-and-generative-models.md)
6. [Generative model foundations](08-generative-models-foundations.md)
7. [Training deep networks in practice](03-training-deep-networks-in-practice.md)
8. [Network design, losses, and diagnostics](04-neural-network-design-losses-and-diagnostics.md)
9. [Optimizers and training dynamics](09-optimizers-and-training-dynamics.md)
10. [Tiny classifier training lab](10-tiny-classifier-training-lab.md)

Work the scalar backprop and convolution examples before using a framework. Then inspect framework tensor shapes and compare one small forward/gradient calculation. Refer back to [calculus and optimization](../01-math/02-calculus-and-optimization.md) for the chain rule, and use the [formula cheatsheet](../01-math/FORMULA_CHEATSHEET.md) for recall.

## Practice

- Complete the [tiny classifier lab](10-tiny-classifier-training-lab.md) and explain why its validation score is not a performance estimate.
- Work through the [deep-learning problem set](PROBLEM_SET_WITH_SOLUTIONS.md) to check tensor shapes, loss gradients, and training diagnosis by hand.
- Implement a tiny feed-forward network and inspect tensor shapes at each layer.
- Compare training and validation curves for underfit and overfit settings.
- Change one training setting at a time and record its effect.
- Compare a neural network against a classical baseline on a small dataset.
- Overfit one tiny batch and verify the loss decreases, then demonstrate why that does not prove generalization.
- Apply a hand-checkable convolution filter and confirm the framework returns the same feature map.
- Visualize attention weights on a short sequence and explain what a causal mask blocks.
- Compare a small CNN with a pretrained model using the same validation data and inference settings.

## Checkpoint project

Train a small image or tabular model. Include a baseline, data-split rationale, learning curves, error examples, a saved configuration, and instructions to reproduce the run.

## Common mistakes to watch for

- Adding layers before checking labels, data leakage, or a simple baseline.
- Treating one successful run as evidence of stable performance.
- Changing many hyperparameters at once and losing track of causes.
- Ignoring compute and memory costs when comparing methods.
