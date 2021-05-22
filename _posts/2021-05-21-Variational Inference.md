---
title: Variational Inference
key: 202105210
tags: Variational, AE, VAE
---

# Variational Inference

One of the core problems of modern statistics is to approximate difficult-to-compute (intractable) probability (usually conditional) densities. There are two main stream methods to solve it: **Monte Carlo sampling (MCMC)** and **Variational inference**. The former focuses on fitting continuous curve through a large amount of discrete values, while the latter employs a tractable simple distribution to approach the true distribution. Let's understand it by an example of **Variational Auto-encoder (VAE)**.

<!--more-->



## Variational Inference on Statistics(VI)

Suppose that we have an intractable conditional density $p(z \vert x)$, and we'd like to construct a well-defined and tractable distribution $q(z \vert x)$ to fit it. The **variational inference (VI)** method optimizes the $q(z \vert x)$ as follows.


$$
q^{\star} = \arg\min KL \left(q(z|x) || p(z|x) \right)
$$


where $KL(\cdot||\cdot)$ means Kullback-Leibler divergence. Let's expand this formula


$$
\begin{align}
KL \left(q(z|x) || p(z|x) \right) &= \sum_{z}-q(z|x)\log\frac{p(z|x)}{q(z|x)} \\
&= \sum_{z}-q(z|x)\log \frac{p(z)p(x|z)}{q(z|x)p(x)} \textit{(Bayesian Variation)} \\
&=\sum_{z}-q(z|x)\log \frac{p(z)p(x|z)}{q(z|x)} + \sum_{z}q(z|x)\log p(x)
\end{align} \\
\Rightarrow \\
\log p(x) = KL \left(q(z|x) || p(z|x) \right) + \sum_{z}q(z|x)\log \frac{p(z)p(x|z)}{q(z|x)}
$$




Now, we have a very inspiring equation consisting of two items. The left-hand side of this equation is not a distribution but a constant (when given $x$), while the right-hand side includes two components, which are our objective function and a variable item, respectively. Therefore, minimizing the objective function is equivalent to maximizing the other item, which is usually denoted as evidence lower bound (ELBO). ELBO can be decomposed into the following forms.


$$
\begin{align}
ELBO &= \sum_{z}q(z|x)\log p(x|z) + \sum_{z}q(z|x)\log \frac{p(z)}{q(z|x)} \\
&= \mathbb{E}_{\sim q(z|x)}\left( \log p(x|z) \right) - KL(q(z|x)||p(z))
\end{align}
$$




The ELMO has a extremely easy-to-optimize formula due to the tractable $q(z \vert x)$ and the feed-forward mathematical expectation of $\log p(x \vert z)$. We will exemplify the utilization of **VI** to empower a simple but useful network architecture **Variational Auto-encoder (VAE)** in the following.



## Auto-encoder (AE)

AE  is designed to pre-train a neural network, which is first introduced by (modular learning in neural networks aaai'1987). Before that, Restricted Boltzmann Machine (RBM) can be viewed as a prototype of AE. RBM is implemented in an undirected graph (Markov network) by minimizing the difference between the marginal distribution of models and data original distributions. It happens that there is a similar case where AE is formulated as a Bayesian version (directed graph). Generally, AE can be computed as:


$$
\begin{align}
z &= f_{enc}(x) \\
\hat{x} &= f_{dec}(z) \\
\mathcal{L} &= | x - \hat{x}  |_2
\end{align}
$$


where $f_{enc}$ and $f_{dec}$ are the encoder and decoder neural networks, respectively. $\mathcal{L}$ is the loss function, which is usually implemented as the mean squared error function.

AE can also be deemed as a unsupervised representation learning from the perspective of dimensionality reduction (DR).



## Variational Auto-encoder (VAE)

Let's review the statistical motivation behind the AE and VAE firstly. AE is designed to formulate the latent representation z of a given sample x. More precisely, AE is based an assumption that observed variable is dependent on the corresponding hidden variables. The only difference between VAE and AE is the hidden space assumption. VAE doesn't regard the hidden representation as a fixed value z (conditional on given x) any longer but a distribution representation $p(z)$. In other words, AE is just **a possible projection** from observable x to hidden z **among a function family** subjected to the VAE. In this view, the VAE makes it possible that we can manually tune hidden variable z over a continuous distribution to obtain continuously varying x, especially controllable attributes of x.

Firstly, VAE employs a **tractable function** (neural network) $q(z \vert x)$ to approximate the **intractable procedure** where we want to figure out the hidden distribution $p(z \vert x)$ of a given (or observed) sample $x$.

Secondly, we reconstruct the observed representation $\hat{x}$ through the given hidden variable $z$, which is obtained by sampling over our tractable approximate distribution.

These two-steps are illustrated in the following figure:

![vae]({{site.url}}/assets/BlogImages/2021-05/VAE.PNG)

In this figure, please note again $q(z \vert x)$ is a neural network, which makes a approximate distribution for intractable relationship $p(z \vert x)$, and $p(x \vert z)$ is just a feed-forward reconstruction function when given sampled $z$.

It is obvious that the aforementioned two-steps **VAE** is just **variational inference (VI)** implemented by neural networks. Now let's uncover the loss function of VAE training, which is just another form of ELBO.


$$
\begin{align}
ELBO_{vae} &= \mathbb{E}_{\sim q(z|x)}\left( \log p(x|z) \right) - KL(q(z|x)||p(z)) \\
&= \mathbb{E}_{\sim x}\left( \log p(\hat{x}|x) \right) - KL(q(z|x)||p(z)) \\
\end{align}
$$


Generally, we assume that **the sampled x and prior of z obey multivariate Gaussian distribution**.


$$
\begin{align}
p(\hat{x}|x) &= e^{-||x - \hat{x}||_2 } \\
p(z) &= \mathcal{N}(\mathbb{0}, \mathbb{I}) \\
q(z|x) &= \mathcal{N} \left( \mu_x, \Sigma_x \right)\\
ELBO_{vae} &= \sum_{x}\left\{-||x-\hat{x}||_2 - KL \left( \mathcal{N} \left( \mu_x, \Sigma_x \right)||\mathcal{N}(\mathbb{0}, \mathbb{I}) \right) \right \}
\end{align}
$$

In this objective function, the first item reflects the **likelihood of reconstruction**, while the second item means **regularization forcing the posterior to approximate the prior.**



During a training step, $\mu_x$ and $\Sigma_x$ are computed by current sample (batch of samples). Specifically, the outputs of encoder network are interpreted as $d + d$ dimensions, which represent the **mean vector and logarithmic diagonal elements** of covariance matrix (uncorrelated assumption with numeric stability).



$$
\mathcal{L}_{vae} = \sum_{x}\left\{ ||x - \hat{x}||_2 + \frac{1}{2}\left(  tr\left( \Sigma_x \right) + \mu_x^T\mu_x-d-\log \det \left( \Sigma_x \right) \right) \right\}
$$


To make the sampling of $z$ from distribution $q(z \vert x)$ is differentiable and the reconstruction loss can be back-propagated to the encoder network with the chain principle. we need use a **reparameterization trick**:


$$
\begin{align}
z \sim q(z|x) &= \mathcal{N} \left( \mu_x, \Sigma_x \right)\\
z &= \mu_x + \epsilon \Sigma_x (\textit{diagonal assumption}) \\
\epsilon &\sim \mathcal{N}(\mathbb{0}, \mathbb{I})
\end{align}
$$


After training of VAE, we can sample different z to generate different x, which is also known as a **deep generative model** and developed into **generative adversarial networks (GAN)**.



## Some notes

- AE maps from a set of x to a set of z, which **is not a distribution but a function** ($f_{enc}(x)$).
- VAE makes a **z's distribution ($q(z \vert x) \approx p(z \vert x)$)** when given x (batch of x) with some constraints, **on which** we can obtain numerous AE-like functions and **reconstruct the input through sampling.**

