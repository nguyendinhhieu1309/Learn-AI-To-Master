# CNNs, Transformers, and Generative Models

## Learning objectives

Recognize common neural architecture families, understand the assumption each makes, and select an appropriate evaluation strategy.

## 1. Convolutional neural networks

Images contain local patterns: edges, corners, textures, and shapes. A convolutional filter slides across an image and applies the same weights at each location. This sharing reduces parameters and encodes the idea that a useful feature can appear in more than one position. Early layers may detect simple patterns; later layers combine them.

Pooling or strided convolutions reduce spatial resolution. This can improve efficiency but may lose precise location information. For segmentation or localization, architectures often preserve or reconstruct spatial detail.

## 2. Transfer learning

A pretrained network has learned useful representations from a large dataset. For a new task, replace the output head and train it, then optionally fine-tune some earlier layers. Use a validation set to choose the strategy. Check that the pretraining data and evaluation set do not overlap in a way that leaks the answer.

## 3. Attention and transformers

An embedding maps a token to a vector. Self-attention lets each token combine information from other tokens. In simplified form, each token creates a query `Q`, key `K`, and value `V`; attention weights are based on query-key similarity and used to combine values:

$$
\mathrm{Attention}(Q,K,V)=\mathrm{softmax}(QK^T/\sqrt{d_k})V.
$$

Transformers stack attention and feed-forward layers. Positional information is needed because attention alone does not encode token order. Transformers work for text, images, audio, and multimodal inputs, but their data and compute requirements can be substantial.

## 4. Generative models

Generative models learn to produce new samples. An **autoencoder** compresses input to a representation and reconstructs it. A **GAN** trains a generator against a discriminator, which can be unstable. A **diffusion model** learns to reverse gradual noise corruption. An autoregressive language model predicts the next token repeatedly. These methods have different objectives and failure modes; “generative” does not mean factually correct or safe.

## 5. Choose by task and constraints

For a small tabular dataset, gradient-boosted trees may be a stronger, cheaper starting point than a neural network. For images, CNNs or vision transformers are common. For sequences with long-range context, transformers are useful. Consider data volume, latency, memory, interpretability, and deployment hardware.

## Practice

Describe the assumption behind convolution (locality and shared features) and attention (content-dependent information mixing). For one task, compare an existing pretrained model to a small baseline and report both quality and inference cost.

## Common mistakes

- Treating architecture names as a substitute for a valid dataset and evaluation.
- Fine-tuning on validation or test examples.
- Comparing models with different input resolution or preprocessing without disclosure.
- Assuming generated outputs are reliable because they look plausible.
