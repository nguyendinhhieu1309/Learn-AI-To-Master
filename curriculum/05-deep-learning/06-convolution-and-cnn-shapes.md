# Convolution and CNN Shapes

Convolutional neural networks (CNNs) use local connections and shared filters to process spatial signals. This lesson computes a 2D convolution-style operation by hand and derives the shapes and parameter counts that commonly cause implementation bugs.

## Learning objectives

- Compute a valid 2D filter response on a small image patch.
- Distinguish a filter, feature map, stride, padding, and dilation.
- Calculate output sizes for common convolution settings.
- Track channels and parameters for a convolutional layer.
- Explain what convolution preserves and what pooling may discard.

## 1. What a convolution layer computes

In deep-learning libraries, the usual operation is technically cross-correlation: the kernel is applied without flipping it. For image coordinate $(i,j)$ and one input channel,

$$
Y_{i,j}=\sum_{a=0}^{K_h-1}\sum_{b=0}^{K_w-1}X_{i+a,j+b}K_{a,b}+b.
$$

The filter weights are shared across every spatial location. A filter that learns an edge detector can respond to that edge wherever it appears in the image. Multiple filters learn multiple output channels.

## 2. Worked 2D example

Let

$$
X=\begin{bmatrix}1&2&0\\0&1&3\\2&1&0\end{bmatrix},
\qquad
K=\begin{bmatrix}1&0\\0&-1\end{bmatrix}.
$$

With stride 1 and no padding, the top-left response is $1(1)+2(0)+0(0)+1(-1)=0$. Sliding right, down, and diagonally gives

$$
Y=\begin{bmatrix}0&-1\\-2&1\end{bmatrix}.
$$

If a bias $b=2$ is used, add 2 to each output. The same kernel weights are reused at all four positions.

## 3. Output size

For one spatial dimension, input length $H$, kernel size $K$, padding $P$, stride $S$, and dilation $D$, the output length is

$$
H_{\mathrm{out}}=
\left\lfloor\frac{H+2P-D(K-1)-1}{S}\right\rfloor+1.
$$

The same calculation applies independently to width. The effective kernel width is $D(K-1)+1$. For $H=32,K=3,P=1,S=1,D=1$, output is 32. With $S=2$, the output is 16. “Same” padding is a framework convention chosen to preserve a target spatial size; verify the framework's behavior for even kernels and stride greater than one.

## 4. Channels and parameter count

An input tensor may be stored as $(B,C,H,W)$ (channels first) or $(B,H,W,C)$ (channels last). Batch $B$, channels $C$, height $H$, and width $W$ are different axes—never infer their meaning only from a number.

For a convolution with $C_{in}$ input channels, $C_{out}$ filters, and kernel $K_h\times K_w$, the number of learned parameters including one bias per filter is

$$
C_{out}(C_{in}K_hK_w+1).
$$

For example, 3 input channels, 16 filters, 3-by-3 kernels, and biases require $16(3\cdot3\cdot3+1)=448$ parameters. The output shape is determined by batch, $C_{out}$, and the two spatial formulas above.

For multiple input channels, each filter contains one kernel per input channel. Their responses are summed and a bias is added. A group convolution restricts which input channels connect to each output channel; depthwise convolution applies one spatial filter per input channel and usually needs a pointwise layer to mix channels.

## 5. Stride, padding, dilation, and pooling

- **Stride** moves the filter multiple pixels per output step, reducing resolution and computation.
- **Padding** adds a border (often zeros) so filters can cover edge pixels and output sizes are controlled.
- **Dilation** spaces kernel elements apart, increasing receptive field without adding learned weights.
- **Pooling** summarizes a local region, often with max or average. It reduces spatial size but may discard exact position.

Downsampling is useful for efficiency but may be harmful when precise localization matters. Detection and segmentation systems use feature pyramids, skip connections, or upsampling paths to recover spatial detail.

## 6. CNN assumptions and architecture

Convolution encodes locality and translation equivariance: shifting the input tends to shift feature maps. It is not complete translation invariance; pooling, data augmentation, or later aggregation can add some tolerance. Local structure is a useful inductive bias for images and many sensor signals, but not every input is naturally a grid.

Receptive field is the region of the original image that can affect a feature. Stacking layers increases it. Residual blocks provide a shortcut path for information and gradients. CNN design must also consider normalization, activation, initialization, augmentation, and the output stride required by the task.

## 7. Debugging a vision model

1. Print the exact input and output shape and identify each axis.
2. Apply a known filter to a tiny hand-checkable image.
3. Confirm pixel scaling, channel order, label mapping, and resize/crop behavior.
4. Overfit a tiny batch and inspect overlays or predictions.
5. Compare validation examples at the same resolution and preprocessing used in deployment.

## Practice with answer checks

1. For input 28, kernel 5, padding 0, stride 1, dilation 1, what is the output length? **Answer:** 24.
2. How many weights including biases for 8 input channels and 32 filters of size $3\times3$? **Answer:** $32(8\cdot9+1)=2,336$.
3. Does a stride-2 convolution usually increase spatial resolution? **Answer:** no; it downsamples.
4. Why share a kernel across locations? **Answer:** it reduces parameters and encodes that a feature may appear in different positions.

## Further reading

- [PyTorch: convolution layers](https://docs.pytorch.org/docs/stable/generated/torch.nn.Conv2d.html)
