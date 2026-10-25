# Deep Learning: Worked Problem Set

Derive each result before using a script as a numerical check. These exercises connect tensor shapes, objectives, gradients, and training decisions.

## 1. Affine layer

A batch has 32 examples with 128 features. A fully connected layer has 64 outputs and bias. Give output shape and parameter count.

### Solution

Weights have shape 128 by 64 and bias has 64 values. Output is 32 by 64. Parameters: 128 × 64 + 64 = 8,256.

## 2. Softmax cross-entropy

For logits [0,0] and target class 2, find probabilities, loss, and gradient with respect to logits.

### Solution

Probabilities are [1/2,1/2]. Loss is log 2. Gradient for one-hot target is p-y=[1/2,-1/2], so gradient descent raises the target logit relative to the other.

## 3. Binary cross-entropy with logits

Target y=1 and predicted probability p=0.8. Find the loss and derivative with respect to logit z.

### Solution

Loss is -log(0.8), about 0.223. For sigmoid plus binary cross-entropy, dL/dz=p-y=-0.2.

## 4. Chain rule

Let h=2x+1, z=3h, and L=(z-5)^2/2. At x=1, compute dL/dx.

### Solution

At x=1, h=3, z=9, so dL/dz=4, dz/dh=3, and dh/dx=2. Therefore dL/dx=4×3×2=24.

## 5. Convolution output

An image is 32 by 32. A convolution uses a 5 by 5 kernel, stride 1, padding 2, and dilation 1. Find output dimensions.

### Solution

$$
H_{out}=\left\lfloor\frac{H+2p-d(k-1)-1}{s}+1\right\rfloor.
$$

Substitution gives floor((32+4-4-1)/1+1)=32. With 16 output channels, channel-first shape is 16 by 32 by 32.

## 6. Receptive field

Two stride-1 convolution layers each use a 3 by 3 kernel with no dilation. What is the receptive-field width?

### Solution

The first sees width 3; the second extends it by 2, yielding width 5. Track feature spacing as well: strides change the spacing between neighboring activations.

## 7. Batch normalization and evaluation mode

Why can predictions change if a model remains in training mode during evaluation?

### Solution

Training-mode batch normalization uses current-batch statistics and updates running estimates; evaluation mode uses stored statistics. Dropout also remains stochastic unless evaluation mode is selected.

## 8. Attention shape

Queries have shape [batch, heads, query_length, head_dim]; keys have [batch, heads, key_length, head_dim]. What shape do scores have?

### Solution

Scores have [batch, heads, query_length, key_length]. A causal mask blocks forbidden future positions. Scaling dot products by sqrt(head_dim) controls their magnitude.

## 9. Residual shape

An activation has shape [batch,64] and a residual branch outputs [batch,128]. Can they be added?

### Solution

No. Project the skip path from 64 to 128, or change the residual branch to output 64. Addition requires compatible dimensions.

## 10. Class imbalance

A dataset has 990 negatives and 10 positives. A classifier predicts negative for every example. Find accuracy and positive recall.

### Solution

Accuracy is 99%; positive recall is 0%. Report class-specific metrics and choose a threshold using validation data and deployment costs.

## 11. Gradient accumulation

The GPU fits batches of 8, but desired effective batch is 32. How many microbatches should be accumulated?

### Solution

Accumulate four microbatches before stepping, scaling each loss by 1/4 if gradients are summed. Batch normalization still sees batches of 8, so this is not identical to a physical batch of 32.

## 12. Learning-rate instability

Loss oscillates and becomes NaN after raising the learning rate. What should be checked before changing architecture?

### Solution

Inspect learning rate, gradient norms, normalization, non-finite data/loss, mixed-precision scaling, and loss reduction. Find the first operation that creates a non-finite value; try a lower learning rate and justified clipping.

## 13. AdamW and L2

Why is decoupled weight decay not necessarily equivalent to adding an L2 penalty with an adaptive optimizer?

### Solution

An L2 penalty adds parameters to the gradient, which is transformed by adaptive moments. Decoupled decay shrinks parameters separately from the adaptive update, so the resulting steps can differ.

## 14. Tiny-batch overfit check

A network cannot drive loss near zero on 12 fixed examples with augmentation off. Name likely bugs.

### Solution

Check input/label alignment, target encoding, output/loss compatibility, gradient flow, optimizer parameter registration, learning rate, accidental detach/no-grad blocks, train/eval mode, and transforms. Inspect predictions and gradients before scaling up.

## 15. Repeated test selection

You try 40 architectures and report the best test accuracy. Why can this be biased if test labels never update gradients?

### Solution

Repeated selection uses test results as feedback. Random variation makes the winner look unusually good. Tune on train/validation and reserve test data for a limited final estimate.

## Review checklist

- Can you calculate tensor shapes and parameter counts by hand?
- Can you derive output-layer gradients for common losses?
- Do train/eval modes use intended normalization and dropout?
- Can you explain optimizer updates?
- Can you debug tiny-batch overfit before scaling training?
- Is model selection separated from final test reporting?
