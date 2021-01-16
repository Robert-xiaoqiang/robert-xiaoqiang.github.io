---
title: Daily Reading
key: 202101140
tags: Reading
---

# Daily Reading

## A Diversity-Promoting Objective Function for Neural Conversation Models

### Summary & Intuitions

- mutual information between source (message) and target (response)
- lack of theoretical guarantee

### Contributions

- decompose formula of mutual information:
  - `anti-lm`: penalize not only high-frequency, generic responses but also grammatical sentence $\rightarrow$ weights of tokens decrease monotonically (early important + lm dominant later)
  - `bidi-lm`: not searching but reranking (generate grammatical sequences and then re-rank them according to the objective of inversed probability)



<!--more-->



### ClarQ: A large-scale and diverse dataset for Clarification Question Generation

### Summary & Intuitions

- collected post and corresponding comments
- improve precision then recall

### Contributions

- down-sampling: confident samples as training set of next iteration
- up-sampling: classifier from $t$ iteration classify samples from $t-1$

