# Attention and Transformers by Hand

Self-attention lets each token create a context-aware representation by selecting and combining information from other tokens. This chapter tracks the query, key, and value shapes, computes a tiny attention example, and explains masks and sequence cost.

## Learning objectives

- Explain query, key, and value in plain language.
- Compute scaled dot-product attention for two tokens.
- Track batch, sequence, head, and feature dimensions.
- Distinguish causal and padding masks.
- Describe a transformer block and its main failure/cost modes.

## 1. Queries, keys, and values

Let $X\in\mathbb{R}^{B\times T\times d_{model}}$ be a batch of $B$ sequences with $T$ tokens, each embedded in $d_{model}$ dimensions. Learned projections create

$$
Q=XW_Q,\qquad K=XW_K,\qquad V=XW_V.
$$

Intuitively, a query asks what information this token needs, a key advertises what information a token contains, and a value carries the information to mix. These are learned projections, not literal natural-language questions.

## 2. Scaled dot-product attention

For one head, similarity scores between every query and key are

$$
S=\frac{QK^T}{\sqrt{d_k}}.
$$

Apply softmax across the key positions for each query, then use the resulting weights to average values:

$$
A=\operatorname{softmax}(S+M),
\qquad
\operatorname{Attention}(Q,K,V)=AV.
$$

$M$ is an optional additive mask; forbidden positions receive a very negative score before softmax so their probability is approximately zero. Dividing by $\sqrt{d_k}$ prevents dot products from growing too large as key dimension increases, which would push softmax into saturated regions.

### Worked two-token example

Suppose one query is $q=(1,0)$ and two keys are $k_1=(1,0)$ and $k_2=(0,1)$. For $d_k=2$, logits are $[1/\sqrt2,0]\approx[0.707,0]$. Softmax gives weights approximately $[0.67,0.33]$. If the values are $v_1=(2,0)$ and $v_2=(0,4)$, the output is

$$
0.67(2,0)+0.33(0,4)\approx(1.34,1.32).
$$

The output mixes information from both positions according to query-key similarity. The weights are content-dependent and change when the query or keys change.

## 3. Multi-head attention and shapes

With $h$ heads, each head uses a lower-dimensional subspace, computes attention separately, and the outputs are concatenated and projected. Common shapes are:

$$
X:(B,T,d_{model}),\quad
Q,K:(B,h,T,d_k),\quad
V:(B,h,T,d_v),
$$

$$
QK^T:(B,h,T,T),\qquad
AV:(B,h,T,d_v).
$$

Usually $d_k=d_v=d_{model}/h$. The quadratic $T\times T$ score matrix is why full self-attention becomes expensive for long sequences. Efficient attention methods change computation or approximation; they do not make context length free.

## 4. Masks and positions

- **Padding mask:** prevents attention to padding tokens that are not real input.
- **Causal mask:** prevents a token from attending to future positions. It is used for next-token generation so the model cannot peek at the answer.
- **Cross-attention mask:** can restrict which encoder/source positions a decoder can read.

Attention alone is permutation-equivariant: rearranging tokens rearranges outputs but does not tell the model which token came first. Transformers therefore add positional information through learned positions, sinusoidal encodings, relative biases, or other schemes. Tokenization and positional representation are part of the model's assumptions.

## 5. What is inside a transformer block?

A common block has multi-head attention, a position-wise feed-forward network, residual connections, and normalization. In simplified notation:

$$
H'=\operatorname{Norm}(H+\operatorname{MHA}(H)),
\qquad
H_{next}=\operatorname{Norm}(H'+\operatorname{FFN}(H')).
$$

The exact order (pre-norm/post-norm), activation (for example GELU/SwiGLU), normalization, and masking vary. Residual paths support gradient flow; normalization stabilizes the representation scale. Decoder-only models use causal self-attention; encoder-decoder models also use cross-attention.

## 6. Training objectives and inference

An autoregressive language model predicts the next token. For sequence tokens $x_1,\ldots,x_T$, the objective is

$$
\mathcal{L}=-\sum_{t=1}^{T}\log P(x_t\mid x_{<t}).
$$

Teacher forcing supplies previous ground-truth tokens during training. At generation time, the model conditions on its own previously sampled or selected tokens, so errors can compound. Sampling strategy (greedy, temperature, top-$k$, nucleus) changes diversity and risk; it does not add factual knowledge.

## 7. Debugging attention

- Verify token IDs, attention masks, padding side, and position indices.
- Check logits and loss on a two-token example.
- Confirm masked attention probabilities are zero and unmasked rows sum to one.
- Inspect whether training and generation use compatible BOS/EOS and chat templates.
- Watch memory with long sequences; attention's score matrix scales approximately as $T^2$.

## Practice with answer checks

1. Along which axis does attention softmax normalize? **Answer:** over key/source positions for each query and head.
2. What does a causal mask block? **Answer:** keys at future positions.
3. Why divide scores by $\sqrt{d_k}$? **Answer:** keep score scale controlled as key dimension grows and avoid overly saturated softmax.
4. Does attention alone encode token order? **Answer:** no; positional information is required.

## Further reading

- [Attention Is All You Need (original paper)](https://arxiv.org/abs/1706.03762)
- [Dive into Deep Learning: attention mechanisms](https://d2l.ai/chapter_attention-mechanisms-and-transformers/index.html)
