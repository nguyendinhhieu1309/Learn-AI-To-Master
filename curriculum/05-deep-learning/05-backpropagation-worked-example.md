# Backpropagation: A Worked Example

Backpropagation is the chain rule applied efficiently to a network's computation graph. This lesson calculates a complete forward pass, loss, gradients, and update for a tiny network before connecting the same ideas to automatic differentiation.

## Learning objectives

- Track activation, weight, and gradient shapes through a network.
- Calculate the forward pass and loss by hand.
- Use the chain rule to send error signals backward.
- Explain why a shared intermediate value accumulates gradient contributions.
- Recognize the limits of hand-derived gradients and when to use autodiff.

## 1. The network and notation

Use one example with two input features, one ReLU hidden unit, and one linear output:

$$
z_h=u_1x_1+u_2x_2+b_h,
\qquad h=\operatorname{ReLU}(z_h),
\qquad \hat y=vh+b_o.
$$

The parameters are $u_1,u_2,b_h,v,b_o$. For a batch of $B$ row examples, $X$ has shape $(B,2)$, hidden weights $(2,1)$ produce $(B,1)$ activations, and the output weight produces $(B,1)$. Keeping the batch axis prevents accidentally mixing examples.

## 2. Forward pass by hand

Take $x_1=1,x_2=2$, target $y=1$, and parameters

$$
u_1=0.1,\;u_2=0.2,\;b_h=0.1,\;v=0.5,\;b_o=0.
$$

The hidden pre-activation and activation are

$$
z_h=0.1(1)+0.2(2)+0.1=0.6,
\qquad h=\operatorname{ReLU}(0.6)=0.6.
$$

The output and half-squared loss are

$$
\hat y=0.5(0.6)+0=0.3,
\qquad L=\frac12(\hat y-y)^2=\frac12(-0.7)^2=0.245.
$$

## 3. Backward pass by the chain rule

Start at the loss and move backward. Let $\delta_o=\partial L/\partial\hat y$:

$$
\delta_o=\hat y-y=-0.7,
\qquad
\frac{\partial L}{\partial v}=\delta_o h=-0.42,
\qquad
\frac{\partial L}{\partial b_o}=\delta_o=-0.7.
$$

The output depends on hidden activation through $v$, so

$$
\frac{\partial L}{\partial h}=\delta_o v=-0.35.
$$

Since $z_h=0.6>0$, the ReLU derivative is 1 at this point, giving $\delta_h=\partial L/\partial z_h=-0.35$. Therefore,

$$
\frac{\partial L}{\partial u_1}=\delta_hx_1=-0.35,
\quad
\frac{\partial L}{\partial u_2}=\delta_hx_2=-0.70,
\quad
\frac{\partial L}{\partial b_h}=\delta_h=-0.35.
$$

The hidden error signal is multiplied by every input that contributed to the pre-activation. For multiple units or paths, a parameter's gradient is the sum of contributions through each path.

## 4. Make one gradient-descent update

With learning rate $\eta=0.1$, subtract the gradient from every parameter:

$$
u_1=0.135,\quad u_2=0.270,\quad b_h=0.135,
\quad v=0.542,\quad b_o=0.070.
$$

The new hidden value is $0.135+0.270(2)+0.135=0.81$ and new prediction is $0.542(0.81)+0.070\approx0.509$. The loss falls to about $0.120$. That confirms this update reduced this example's loss; it says nothing by itself about performance on unseen examples.

## 5. General layer equations

For row-major batches, layer $l$ computes

$$
Z^{(l)}=A^{(l-1)}W^{(l)}+b^{(l)},
\qquad A^{(l)}=\phi(Z^{(l)}).
$$

If batch size is $B$, input width is $d_{l-1}$, and output width is $d_l$, then

$$
A^{(l-1)}:(B,d_{l-1}),\quad
W^{(l)}:(d_{l-1},d_l),\quad
Z^{(l)}:(B,d_l).
$$

Given a gradient with respect to the next activation, backprop multiplies by the derivative of the activation and then by the appropriate transposed weight matrix. For a final affine layer with batch-averaged loss and error matrix $\Delta^{(l)}=\partial L/\partial Z^{(l)}$:

$$
\frac{\partial L}{\partial W^{(l)}}=(A^{(l-1)})^T\Delta^{(l)},
\qquad
\frac{\partial L}{\partial b^{(l)}}=\sum_{i=1}^{B}\Delta_i^{(l)}.
$$

The exact reduction over batch depends on whether the loss was summed or averaged. This is why loss reduction and gradient scale belong in experiment notes.

## 6. Automatic differentiation and gradient checks

Frameworks build a computation graph and apply chain rules automatically. Autodiff does not know whether your model or target is conceptually correct: it faithfully differentiates whatever program you wrote. Check shapes, loss values, and finite gradients before large runs.

For a custom operation, compare one analytic gradient against a central finite difference on a tiny double-precision input:

$$
g_j^{\mathrm{numeric}}=
\frac{L(\theta+\epsilon e_j)-L(\theta-\epsilon e_j)}{2\epsilon}.
$$

Avoid checking exactly at a non-differentiable point such as ReLU input zero. Finite differences are a debugging tool, not a scalable training method.

## 7. Common gradient problems

- **Vanishing gradients:** repeated derivatives smaller than one shrink signals in earlier layers.
- **Exploding gradients:** repeated factors larger than one make gradient magnitudes grow.
- **Dead ReLU unit:** activation stays negative, giving zero gradient through that unit.
- **Wrong objective or labels:** gradients may be mathematically valid for an unintended target.
- **Incorrect reduction or mask:** gradients include padding or examples that should have been excluded.

Inspect gradient norms by layer, output/target shapes, and a tiny-batch overfit. Clipping can limit extreme gradients but does not fix a broken data or loss definition.

## Practice with answer checks

1. If output error $\delta_o=-0.7$ and activation $h=0.6$, what is $\partial L/\partial v$? **Answer:** $-0.42$.
2. If a ReLU pre-activation is negative, what gradient passes through that unit? **Answer:** zero (away from the kink at zero).
3. Why is the bias gradient a batch sum? **Answer:** the shared bias contributes to every example's pre-activation, so its per-example contributions add (then divide if the loss is averaged).
4. Can autodiff detect that labels were paired with the wrong inputs? **Answer:** no; it only differentiates the program it receives.
