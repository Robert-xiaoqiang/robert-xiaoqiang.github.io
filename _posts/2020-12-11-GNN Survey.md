---
title: GNN Survey
key: 202012110
tags: GNN, Survey
---

# GNN Survey

## Overview

  In my opinion, graph neural networks is worth exploring within research field, not engineering field, especially large scale application. Anyway, this survey is a naive summarization of my recent reading. Writing in English is just a practice of my poor writing ability.

  Without whistles and bells, there are some equivalent concepts or notions in graph terminology. First of all, graph signal is general representation, which stands for collection of node embeddings or node signals. Graph Fourier Transformation (GFT) is just discrete version Fourier Transformation (FT). Fourier Transformation (FT) converts temporal signals to frequency signal with the help of operator $e^{-iwt}$ (sine, cosine base function), and vice versa. In a similar way, GFT converts spatial domain to spectral domain with the help of $\phi^T$ (eigen base vector).

Note **spatial domain** is also known as **vertex domain** or **data space domain**. In the contrast, **Spectral domain** is also denoted as **feature space domain**



<!--more-->



  More straightforwardly, GFT converts node function(signal) to an eigen function(signal). Here **node function** represents a node-wise function, which means every node has a value (scalar or multi-dimensional feature (tensor)), shaped $n_1 \times d$ . eigen function denotes a eigen-wise function, which means every eigen value (or vectors) has a value (scalar or multi-dimensional feature (tensor)), shaped $n_2 \times d$ . Note we assume that $n_1 == n_2 == \#eigen \, vectors == \#eigen \, values$, because the initial kernel graph construction is based on data matrix, which is usually full-rank due to dataset variance.

  It is easy to find that we obtain a new node representation (embedding) by aforementioned GFT. However that transformation utilizes graph global structure (because of Laplacian or Adjacency matrix) to embed own nodes. Using this determinism-will idea, we can adjust embeddings by manual setting or learnable manner, which is also named as **spectral-based** method and the best practice is Graph Convolutional Networks (GCN). In this article, I call it **global view** embedding, details in [section 2](#Global View).

  Apparently, global view embeddings is limited to undirected graph (due to decomposition of real symmetric matrix) and strongly corelated to node ordering. So operating embeddings on **vertex domain** directly is desirable. This kind of method is named **spatial-based** method and the best practice is Message Passing Protocol (MPP) or Recurrent Graph Neural Networks (RecGNN). In this article, I call it **local view** embedding, details in [section 3](# Local View).

  So far, we have solved node embedding problems in both **global and local** view. We can also conduct graph embedding and time-dependent graph data, details in [section 4](Embedding View) and [section 5](Spatial-Temporal View).

  Some notations or denotations are as follows:



| Notations | Descriptions                |
| --------- | :-------------------------: |
| $G$       | a graph                     |
| $V$       | the set of nodes in a graph |
| $v$       | a node $v \in V$            |
| $E$       | the set of edges in a graph |
| $e_{i,j}$ | an edge $e_{i,j} \in E$     |
| $A$ or $W$ | the graph adjacent matrix    |
| $X \in R^{n \times d}$ | node feature matrix |
| $x^{v} \in R^{d}$ | feature of node $v$ |
| $X^e \in R^{n \times c}$ | edge feature matrix |
| $x^{e}_{(v,u)} \in R^{c}$ | feature of edge $e$ between $v$ and $u$ |
| $n$ | $n = |V|$ the number of nodes |
| $m$ | $m = |E|$ the number of  edges |
| $d$ | the dimension of node feature |
| $h$ | the hidden dimension of node feature |
| $c$ | the dimension of edge feature |
| $L$ | Laplacian eigen map |
| $\mu_i, \lambda_i$ | $i$-th eigen vector and eigen value of Laplacian matrix |
| $\phi$ | $\phi = [ \mu_1, \cdots, \mu_n]$  Graph Fourier Inverse Transformation Operator |

## Global View

Let's begin the extraordinary adventure with global view embedding. Please keep it in your mind that global view is globality of original data (**spatial domain**)

## Local View

## Embedding View

## Spatial-Temporal View

