---
title: QG & LM Survey
key: 202101060
tags: QG LM Survey
---

# QG & LM Survey

## Overview

This article is mainly based on the question generation papers from ACL'20. However, there are quite a few papers from previous years due to recurssive reading process. Indeed I know little about history of language models and typical NLP tasks. Let's step forward by defining some notations or denotations. As we know, the unit of NLP is tokens or token embeddings. Note a token doesn't mean a word actually, though we can regard a word as a printablt token. That means there are some non-printable tokens, i.e. Classifier tag, Separator tag, Begin of Sequence, Start of Sequence and End of Sequence et al. . Printability and non-printability are the notions from ASCII codes (i.e. 31 non-printable + 95 printable + 1 non-printable). By the way, It is obvious that ASCII characters, Unicode characters or any available single-width characters belong to printable tokens. In addition to token, another core concept is sequence. Because a single token scarcely occures alone, it co-occures with other tokens based on ordering or temporal depenency, which is also the key challenge of modeling natural languages. Therefore, **sequence embeddings** (the list of token embddings in this sequence) is the content of data flow in NLP tasks. On the contrary, **feature maps** (the collection of multiple channels of image features) is the counterpart in CV tasks. Besides, like **croping and resizing** to procure well-shaped batch data in CV tasks, we need **truncating or padding** to process too long or too short sentences. Padding operation, however, inevitably compute extra gradients to the input text during back propagation. Conventionally we feed a mask with the input text to specify the gradientless span within the text, which also has an ambiguous and confusing name **attention_mask** in **Transfromer** based architectures.

<!--more-->



## Language Models

There are many lanuage models (LM) since the success of deep neural networks in NLP, especially the extraordinary performance of pre-training + fine tuning scenario. Without bells and whistles, LM just models the occurrence probablity of observed sequence (or sentence). In terms of probablity theory, optimization of LM is to maximize the likelihood of joint distribution of tokens (or words).

### word2vec (2013)

There are quantities of training tricks (negative sampling) as well as skip-gram and CBOW training strategy. 

