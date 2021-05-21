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
The term "self-supervised learning" is first introduced in robotics, where trainin data is automatically labeled by leveraging the raltions between different input sensor signals. In the invited speech on AAAI 2020, the turing award winnner Yann LeCun described SSL as: **the machine predicts any parts of its input for any observed part**:
   - Obtain "labels" from the data itself by using a "semi-automatic" process;
   - Predict part of the data from other parts;
where "other parts" could be incomplete, transformed, distorted, or corrupted. In other words, "predict" can also be regarded as "recover". In an extreme situation, the hidden/unobserved and unhidden/observed parts are likely to the whole (original) data.


###### Difference between SSL and unsupervised learning
Firstly, the **supervised learning** must be reduced into **classification** and **regression** according the label attributes (discrete vs. real value or categorical vs. continuous variable), especially image-level, pixel-level, text-level and token level.
Secondly, the **semi-supervised learning** aims to incorporate unlabeled data into the supervised algorithm. They are usually implemented by consistency regularization under the hypothesis of distribution/maniford/entropy.
Finally, the **unsupervised learning** is a kind of algorithm without external signals (manual labels) to guide the target task. They usually serve as a preprocessing procedure, such as dimensionality reduction (DR), clustering, community discovery and anomaly detection. This kind of algorithm is designed to mine/capture the specific data patterns through different heuristic priors or constraints.
From the aforementioned concepts, **self-supervised learning (SSL)** can be viewed as a branch of **unsupervised leaerning**. In this branch, there the 2 following characteristics: (i) SSL follows the paradigm of **pre-training then finetuning** and detects the general data pattern, which is useful for lots of downstream tasks, especially multiple modalities tasks (i.e. **multimedia SSL**). (ii) The downstream tasks are usually supervised learning algorithm, and SSL concentrates more on learning a robust and generalizable representation. In this viewpoint, **SSL is equivalent to unservised representation learning**.


##### Theory Guarantee





