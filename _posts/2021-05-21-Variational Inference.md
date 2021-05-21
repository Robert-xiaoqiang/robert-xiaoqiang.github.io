---
title: Variational Inference
key: 202105210
tags: Variational, AE, VAE
---

####  Variational Inference

One of the core problems of modern statistics is to approximate difficult-to-compute (intractable) probability (usually conditional) densities. There are two main stream methods to solve it: **Monte Carlo sampling (MCMC)** and **Variational inference**. The former focuses on fitting continuous curve through a large amount of discrete values, while the latter employs a tractable simple distribution to approach the true distribution. Let's understand it by an example of **Variational Auto-encoder (VAE)**.

<!--more-->



#### Variational Inference (VI)

Suppose that we have an intractable conditional density $p(z|x)$, and we'd like to construct a tractable distribution $q(z|x)$ to fit it. The **variational inference (VI)** method optimizes the $q(z|x)$ as follows.







##### Auto-encoder (AE)

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



##### Variational Auto-encoder (VAE)

The only difference between VAE and AE is the hidden space assumption. VAE doesn't regard the hidden representation as a fixed value (conditional on given x) any longer but a distribution representation.