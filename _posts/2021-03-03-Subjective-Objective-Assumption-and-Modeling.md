---
title: Subjective, Objective, Assumption and Modeling
key: 202103030
tags: MLE, Bayesian
---

# Subjective, Objective, Assumption and Modeling

## Overview

There are 2 main methods to estimate parameters in statistics. i.e. **frequency-based method (Maximum Likelihood Estimation)** and **Bayesian-based method (Bayesian Estimation)**. Frankly speaking, we are supposed to get a deep insight to it and have a intuitive understanding on estimation. In this note, I provisionally offer some dimensions to explore it. i.e. **motivation, theoretical guarantee and algorithm**. Maybe I will enrich it in my future study. Note that there is **just a difference** in modeling unknown thing between 2 methods, but both are statistical method, which aims to estimate the whole distribution from the sampling data (in some case, do hypothesis then verify it).



<!--more-->



## Motivation

- **frequency-based method** is based on the assumption or belief that unknown thing is **a fixed (or constant) value** and we just don't know it, but we are able to fit (or simulate) it by quantities of experiments (of course, more experiments, more confident). **In other words, it is subjective and models unknown thing by uncertainty.**

- **Bayesian-based method** is a application of **Bayesian Formula** in parameter estimation. Its key idea is that unknown thing **is not fixed and obeys some distribution**. Obviously, that distribution is also driven by some **descriptive parameters** (e.g. mean and variance). **In other words, it is objective and models unknown thing by incompleteness (due to non full-fledged observations)**



## Algorithm

- Given observations $D$ (i.i.d. experiments) and unknown parameter $\theta$, **frequency-based method (MLE)** maximizes the unknown belief from known experiments. That is, if it happens, it is more likely to happen once again. In detail, we maximize the probability of observed events $D$ conditioned on the parameter $\theta$.

$$
\theta^\star = \mathop{\arg\max}_\theta \{P(D|\theta)\}
$$

- In the perspective of **Bayesian-based method**, our goal is not to find a fixed parameter. On the contrary, we need delineate the distribution behind the parameter, because of **the aforementioned key idea of Bayesian-based method**. Specifically, $\theta$ is driven by a distribution, namely **posterior distribution $P(\theta|D)$**. According to **Bayesian Formula**:
  $$
  P(\theta|D) = \frac{P(D|\theta)P(\theta)}{P(D)} = \frac{P(D|\theta)P(\theta)}{\int_\theta P(D|\theta)P(\theta) d\theta}
  $$
  In the practical case, there are 2 kinds of approximation:

  - In order to obtain a fixed value from a posterior distribution, we can force the posterior to be maximum (**Maximum A Posterior**), which is denoted as:
    $$
    \begin{align}
    \theta^\star &= \mathop{\arg\max}_\theta \{P(\theta|D)\} \\
    &= \mathop{\arg\max}_\theta\{ \frac{P(D|\theta)P(\theta)}{\int_\theta P(D|\theta)P(\theta) d\theta} \} \\
    &= \mathop{\arg\max}_\theta\{P(D|\theta) P(\theta)\}
    \end{align}
    $$
    Compared to MLE, the objective (after logarithm) has one more item than it. we can regard the extra item as regularization w.r.t. $\theta$ , which is from **the prior distribution $$P(\theta)$$**. Note that this kind of approximation is totally dependent on the assumption of prior.

  - A stronger approximation is that posterior has the same distribution as prior, namely **Conjugate Distribution**. The prior is regarded as **conjugate prior** w.r.t. likelihood $P(D|\theta)$.

## Theoretical background

- MLE maximizes the belief, so it relies on experiments heavily. In other words, more experiments, more confident. (证明见：霍夫丁不等式， 大数定律)
- MAP is completely based on Bayesian Formula, which is just a classic viewpoint of Bayesian Party. (类似的：万物皆分布， 皆由分布背后的参数驱动， 例：贝叶斯神经网络)

