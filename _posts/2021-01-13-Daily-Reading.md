---
title: Daily Reading
key: 202101130
tags: Reading
---

# Daily Reading

##  Tensor2Tensor: One Model to Learn to Them All

### Summary & Intuitions

- multi-modality multi-task learning
- modality-specific subnets: typical pipelines
- modality-agnostic body: separable convolution (row conv + depth-wise conv, due to 1-d, 2-d inputs) and     attention mechanism (self-attended + Query-presence-attended)
- joint training of tasks with deficient and sufficient data

### Contributions

- engineering considerations of modality subnets
  - language input: linear mapping
  - language output: linear mapping + `softmax` 
  - image input: 2 separable conv + 1 pooling + residual link
  - categorical output: 3 separable conv + 1 pooling + residual link + 2 separable conv + GAP + linear
  - Audio input and output: wave (1d) or spectrogram (2d) is the same as aforementioned image input



<!--more-->



## Outrageously Large Neural Networks: The Sparsely-Gated Mixture-Of-Experts Layer

### Summary & Intuitions

- weight matrices based experts
- block based dropout
- learnable dropout and expert contribution weight

### Contributions

- noisy top-k `softmax` gating network outputs sparsity vector, which is used for weighted sum and choose experts (i.e., 0 value of one component means absence of associated expert)
- large scale with thousands of experts 



## Mixture Content Selection for Diverse Sequence Generation

### Summary & Intuitions

- hard mixture-of-expert with 0, 1 selective vector
- separate selection and generation, because joint training need intractable enumeration
- training by heuristic focus and testing by sampling focus (i.e., guidance as the target of selector during training)

### Contributions

- modeling latent focus explicitly by Bi-GRU + Bernoulli distribution, which has same length as the input sequence
- E-step: select expert have best fitting to guidance
- M-step: concatenate sequence embedding with focus vector of the best expert and generate sequence as usual
- guidance design ranges from word stem occurrence to minimal cover according to specific generation task