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

  So far, we have solved node embedding problems in both **global and local** view. We can also conduct **graph embedding** and **time-dependent graph** data, details in [section 4](Embedding View) and [section 5](Spatial-Temporal View).

  Some notations or denotations are as follows:



| Notations | Descriptions                |
| --------- | :-------------------------: |
| $G$       | a graph                     |
| $V$       | the set of nodes in a graph |
| $v$       | a node $v \in V$            |
| $N(v)$ | set of neighbor nodes of $v$ |
| $E$       | the set of edges in a graph |
| $e_{ij}$ | an edge $e_{ij} \in E$     |
| $A$ or $W$ | the graph adjacent matrix    |
| $X \in R^{n \times d}$ | node feature matrix |
| $x^{v} \in R^{d}$ | feature of node $v$ |
| $X^e \in R^{n \times c}$ | edge feature matrix |
| $x^{e}_{(v,u)} \in R^{c}$ | feature of edge $e$ between $v$ and its neighbor $u$ |
| $n$ | $n = |V|$ the number of nodes |
| $m$ | $m = |E|$ the number of  edges |
| $d$ | the dimension of node feature |
| $h$ | the hidden dimension of node feature |
| $c$ | the dimension of edge feature |
| $L$ | Laplacian eigen map |
| $\mu_i, \lambda_i$ | $i$-th eigen vector and eigen value of Laplacian matrix |
| $\phi$ | $\phi = [ \mu_1, \cdots, \mu_n]$  Graph Fourier Inverse Transformation Operator |

## Global View

Let's begin the extraordinary adventure with global view embedding. Please keep it in your mind that global view is globality of original data (**spatial domain**), which means **spectral domain** is not dependent or orthogonal to **spatial domain**, but different views of the same data.

Given a undirected graph $G$, we firstly obtain Laplacian Eigen Map $L = D - A$, where $D$ is degree diagonal matrix. Specifically, $D = \{d_{ij}\}$ and $d_{ii} = \sum_{k = 1}^{n}A_{ik} = \sum_{k=1}^{n}A_{ki}$, $d_{ij} = 0$ when $i \ne j$, $A$ is adjacency matrix with $A_{ii} = 0$. Although there are modified Laplacian Eigen Map (such as normalized Laplacian, random walk Laplacian), we simply use the vanilla version.

Decomposing $L$ and we can obtain eigen vectors $\mu_i$ and associated eigen values $\lambda_i$, suppose that $\lambda_1 < \lambda_2 < \cdots < \lambda_n$:
$$
\phi^{-1}L\phi = \Lambda \\
where \, \phi = [ \mu_1, \cdots, \mu_n], \, \Lambda = 
\left[\begin{matrix}
\lambda_1 & 0 & \cdots & 0 \\
0 & \lambda_2 & \cdots & 0 \\
\vdots & \vdots & \ddots & \vdots \\
0 & 0 & \cdots & \lambda_n
\end{matrix}\right] \\
and \, \phi^{-1} = \phi^T, orthogomal \, matrix
$$
Now, let's find some interesting properties of $L$. we simply suppose that the dimension of node feature is 1. Note when it is more than 1, we can conduct the following procedure  in a **predictor-wise**  manner to get the same perspective, thus $X \in R^n$ and $x_i$ is a scalar feature of node $i$. In addition, we suppose the graph is constructed from hard kernel (limitation of soft kernel graph) , thus $A \in \{ 0, 1\}^{n \times n}$.
$$
\begin{align}
LX &= (D-A)X \\
&= \left[\begin{matrix}
x_1d_{11} \\
x_2d_{22} \\
\vdots \\
x_nd_{nn}
\end{matrix}\right]  - 
\left[\begin{matrix}
x_{1i} + x_{1j} + \cdots \\
x_{2i} + x_{2j} + \cdots \\
\vdots \\
x_{nj} + x_{nj}  + \cdots\\
\end{matrix}\right] \\
&where \, e_{1i} \, e_{1j} \, e_{2i} \, e_{2j} \, e_{ni} \, e_{nj} \in E \\
&= \left[\begin{matrix}
\sum_{j \in N(1)}(x_1 - x_j) \\
\sum_{j \in N(2)}(x_2 - x_j) \\
\vdots \\
\sum_{j \in N(n)}(x_n - x_j)
\end{matrix}\right]
\end{align}
$$


It is obvious

- that **Laplacian Operator (Laplacian Eigen Map)** represents the **difference (or variation)** between every node and its neighbors.
- that decomposition of Laplacian Eigen Map $L\mu = \lambda\mu$ has trivial solutions when $\lambda = 0$ (I will show $\lambda \ge 0$ next paragraph) and the number of trivial solutions (the multiplicity of the smallest eigen value $0$ ) is just the number of connected components (Note there is one trivial solution at least , that is with the same value for all nodes), because we can assign a fixed value for every connected components and make every node in this component with this value. **Same or not same** value(s) for different connected components is dependent on the specific situation.
- that **spectral clustering** is a method for clustering ( dimensionality reduction and clustering), which just partitions the low-frequency base signals (regarding eigen vectors corresponding to **the second least** eigen value as reduced results).

Let's take one more step in a $n$-element quadratic form or standard form:
$$
\begin{align}
    X^TLX &= \left[\begin{matrix}
    x_1 & x_2 & \cdots x_n
    \end{matrix}\right] \left[\begin{matrix}
    \sum_{j \in N(1)}(x_1 - x_j) \\
    \sum_{j \in N(2)}(x_2 - x_j) \\
    \vdots \\
    \sum_{j \in N(n)}(x_n - x_j)
    \end{matrix}\right] \\
    &= \sum_{e_{ij} \in E}{(x_i - x_j)^2}
\end{align}
$$
It is apparent

- that $L$ is a semi-positive definite matrix, and least eigen value $\lambda_1 = 0$ and I have proof its trivial solution s ahead of time.
- Both $LX$ and $X^TLX$ represent the difference or variation between every node and its neighbors. we call the latter as **Total Variation**.
- go one more step



## Local View

## Embedding View

## Spatial-Temporal View

