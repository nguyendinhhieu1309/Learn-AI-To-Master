# Generative Models: Foundations

Generative models learn a distribution or a procedure for producing samples. The word “generative” describes the modeling goal; it does not mean the result is true, useful, original, or safe.

## Learning objectives

- Distinguish discriminative prediction from generative modeling.
- Compare autoregressive, variational-autoencoder, GAN, and diffusion objectives.
- Explain training versus sampling for each family.
- Identify common evaluation gaps and deployment risks.

## 1. Discriminative and generative questions

A discriminative model estimates $P(y\mid x)$ or a decision boundary: given an image, what class is present? A generative model aims to represent $P(x)$ or $P(x\mid c)$ and sample plausible data. Some models combine both aims. A likelihood or training score is not the same as human-perceived quality or downstream utility.

## 2. Autoregressive models

The chain rule factors a sequence distribution into next-step conditionals:

$$
P(x_1,\ldots,x_T)=\prod_{t=1}^{T}P(x_t\mid x_{<t}).
$$

Training commonly minimizes next-token negative log-likelihood. Sampling proceeds sequentially: predict a distribution for the next token, select/sample a token, append it, and repeat. This is flexible but inference is sequential, and outputs can drift as the model conditions on its own earlier mistakes.

## 3. Variational autoencoders (VAEs)

An encoder approximates a latent posterior $q_\phi(z\mid x)$ and a decoder models $p_\theta(x\mid z)$. Training maximizes the evidence lower bound (ELBO):

$$
\log p_\theta(x)\geq
\mathbb{E}_{q_\phi(z\mid x)}[\log p_\theta(x\mid z)]
-D_{KL}(q_\phi(z\mid x)\|p(z)).
$$

The first term rewards reconstruction; the KL term encourages the encoded distribution to stay near a prior so new samples can be generated from that prior. A standard normal prior is common but not mandatory. VAEs provide a structured latent-space objective, though pixel-level likelihood may not align with perceived sharpness or semantic quality.

## 4. Generative adversarial networks (GANs)

A generator maps noise $z$ to sample $G(z)$. A discriminator attempts to distinguish real from generated examples. Their adversarial objectives can produce sharp samples, but training may be unstable, mode collapse can reduce diversity, and the discriminator can become too strong or too weak. Monitor both sample quality and coverage/diversity, not one pleasing image.

## 5. Diffusion models

Forward diffusion gradually adds noise to data. A neural network learns to predict or remove noise at different time steps. Sampling starts with noise and repeatedly applies a learned reverse process:

$$
x_T\sim\mathcal{N}(0,I),
\qquad x_{t-1}=\text{reverse-step}_\theta(x_t,t).
$$

Many systems train with a noise-prediction objective such as $\mathbb{E}\|\epsilon-\epsilon_\theta(x_t,t)\|_2^2$. Conditioning can guide generation with text, class, or other input. Sampling often requires multiple denoising steps; quality, speed, and diversity depend on model, sampler, and guidance settings.

## 6. Compare model families

| Family | Typical training target | Sampling pattern | Frequent challenge |
|---|---|---|---|
| Autoregressive | Next-token likelihood | Sequential tokens | Sequential latency and error accumulation |
| VAE | ELBO / reconstruction plus KL | Sample latent, decode | Blurry outputs or latent mismatch |
| GAN | Adversarial game | One generator pass | Instability and mode collapse |
| Diffusion | Noise/score prediction | Iterative denoising | Sampling cost and conditioning artifacts |

These are broad patterns, not complete definitions; modern systems combine ideas and objectives.

## 7. Evaluation beyond a sample gallery

Evaluate against the intended use: perceptual quality, diversity, constraint adherence, latency, controllability, privacy, downstream utility, and failure rates. Automated distribution metrics can correlate poorly with human judgments or conceal rare failures. Use representative prompts/conditions, human review where appropriate, and clear documentation of known limits.

## 8. Data, consent, and safety

Training data can contain private, copyrighted, sensitive, or harmful content. Generated outputs may memorize, imitate, expose, or misrepresent information. Record data provenance and permissions, test for memorization and disallowed content, add provenance controls where relevant, and provide a human review path for high-impact use.

## Practice

1. Why is next-token prediction an autoregressive objective?
2. Which VAE term encourages encoded latents to remain compatible with the prior?
3. What does mode collapse mean in a GAN?
4. Why is one high-quality generated example insufficient to demonstrate a model is reliable?

**Answer checks:** each prediction conditions on the previous prefix; the KL divergence term; the generator produces limited diversity/modes; it says nothing about diversity, reliability, failure frequency, or performance across conditions.

## Further reading

- [Dive into Deep Learning: generative deep learning](https://d2l.ai/chapter_generative-models/index.html)
- [PyTorch: DCGAN tutorial](https://pytorch.org/tutorials/beginner/dcgan_faces_tutorial.html)
- [Ho et al., Denoising Diffusion Probabilistic Models](https://arxiv.org/abs/2006.11239)
