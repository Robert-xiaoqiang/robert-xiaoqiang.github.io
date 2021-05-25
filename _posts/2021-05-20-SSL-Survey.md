---
title: SSL Survey
key: 202105200
tags: Self-supervised Learning, Unsupervised Learning
---

#### SSL Survey

We are on the cusp of a major research revolution, which is made by deep learning. In my perspective, there two outstanding contributions on the network architecture in this revolution. They are $\textit{ResNet}$ and $Transformer$. As the research exploration continues to deepen and especially the increase of computational capacity, the technology using unlabeled data attracts more and more concentrations. There is no doubt that the **self-supervised learning (SSL)** is a direction deserve diving into and a general methodology contribution in the revolution. Therefore, this post will focus survey the cutting-edge development of SSL from the following aspects: theory guarantee, image SSL, sequence SSL and graph SSL.

<!--more-->

##### Definition

###### Origination
The term "self-supervised learning" is first introduced in robotics, where training data is automatically labeled by leveraging the relations between different input sensor signals. In the invited speech on AAAI 2020, the Turing award winner Yann LeCun described SSL as: **the machine predicts any parts of its input for any observed part**:
   - Obtain "labels" from the data itself by using a "semi-automatic" process;
   - Predict part of the data from other parts;
where "other parts" could be incomplete, transformed, distorted, or corrupted. In other words, "predict" can also be regarded as "recover". In an extreme situation, the hidden/unobserved and unhidden/observed parts are likely to the whole (original) data.


###### Difference between SSL and unsupervised learning
Firstly, the **supervised learning** must be reduced into **classification** and **regression** according the label attributes (discrete vs. real value or categorical vs. continuous variable), especially image-level, pixel-level, text-level and token level.
Secondly, the **semi-supervised learning** aims to incorporate unlabeled data into the supervised algorithm. They are usually implemented by consistency regularization under the hypothesis of distribution/manifold/entropy.
Finally, the **unsupervised learning** is a kind of algorithm without external signals (manual labels) to guide the target task. They usually serve as a preprocessing procedure, such as dimensionality reduction (DR), clustering, community discovery and anomaly detection. This kind of algorithm is designed to mine/capture the specific data patterns through different heuristic priors or constraints.
From the aforementioned concepts, **self-supervised learning (SSL)** can be viewed as a branch of **unsupervised learning**. In this branch, there the 2 following characteristics: (i) SSL follows the paradigm of **pre-training then finetuning** and detects the general data pattern, which is useful for lots of downstream tasks, especially multiple modalities tasks (i.e. **multimedia SSL**). (ii) The downstream tasks are usually supervised learning algorithm, and SSL concentrates more on learning a robust and generalizable representation. In this viewpoint, **SSL is equivalent to unsupervised representation learning**.


##### Theory Guarantee




##### Generative methods
There are four main major approaches in this kind of SSL, which consists of **Auto-regressive (AR)**, **Auto-encoder (AE)**, **Flow-based** and **Hybrid models**.

##### Auto-regressive (AR)

This series of algorithms focus on modeling the generation procedure by a sequence. Specifically, they firstly regard the **observed sample** as a **sequence of variables** (a **temporal series**), then maximize the likelihood of observed sequence.  During the computation of likelihood, the **joint distribution** of the sequence can be decomposed into the multiplication between **marginal and conditional distributions** by applying Bayesian Formula step-by-step, which can be represented as:


$$
\begin{align}
\theta^{\star} &= \arg\max \log \prod_{i}^N p(x_i \mid \theta) \\
&= \arg\max \log \prod_{i}^N p(x_i^1, \cdots, x_i^t, \cdots, x_i^T \mid \theta) \\
&= \arg\max \log \prod_{i}^N \prod_{t}^T p(x_i^t \mid x_i^1, \cdots, x_i^{t-1}, \theta) \\
&= \arg\max \sum_{i}^N \sum_{t}^T \log p(x_i^t \mid x_i^1, \cdots, x_i^{t-1}, \theta)
\end{align}
$$


where $\log p(x_i^t \mid x_i^1, \cdots, x_i^{t-1}, \theta)$  means the observed probability of the $t$-th variable in the $i$-th observed sample, which is conditioned on the previous variables ($1, \cdots, t-1$) of the identical sample. Note that although the decomposition changes with the selection of the first variable, **the AR model has an unique one because of the temporal dependency.** In addition, this objective function is also employed to optimize the **language model (LM)** and renamed as perplexity loss. Both **perplexity loss in LM** and cross entropy loss in classification can be converted into **maximum likelihood estimation (MLE).**

Different SSL AR models vary in the sequential modeling and utilization of outputs:



| Model     | Fields | Sequential Modeling                                   | Utilization of Outputs               | Comments                                                     |
| --------- | ------ | ----------------------------------------------------- | ------------------------------------ | ------------------------------------------------------------ |
| GPT/GPT-2 | NLP    | Trivial                                               | Next sentence prediction (NSP)       | Similarities: Transformer decoder, Masked self-attention <br />Differences: GPT (pre-training+finetuning), GPT-2 (in-context learning, given both inputs and the task) |
| PixelCNN  | CV     | top-left to bottom-right by masked filter/kernel      | Next pixel prediction (NPP)          | Embed PixelCNN as the decoder of Auto-encoder to perform SSL |
| PixelRNN  | CV     | top-left to bottom-right <br />R channel to G channel | Next pixel channel prediction (NPCP) | convolution -> long/short memory modeling <br />neighbor strategy (many $h_{t-1}^0$ to be considered) <br />Row LSTM (3 pixels in last row), Diagonal BiLSTM (4/8 adjacency) |

