---
title: GNN Survey
key: 202012110
tags: GNN, Survey
---

[TOC]

# GNN Survey

## Overview

  In my opinion, graph neural networks is worth exploring within research field, not engineering field, especially large scale application. Anyway, this survey is a naive summarization of my recent reading. Writing in English is just a practice of my poor writing ability.

  Without whistles and bells, there are some equivalent concepts or notions in graph terminology. First of all, graph signal is general representation, which stands for collection of node embeddings or node signals whatever domain we talk about. Graph Fourier Transformation (GFT) is just discrete version Fourier Transformation (FT). Fourier Transformation (FT) converts temporal signals to frequency signal with the help of operator $e^{-iwt}$ (sine, cosine base function), and vice versa. In a similar way, GFT converts spatial domain to spectral domain with the help of $\phi^T$ (eigen base vector).

Note **spatial domain** is also known as **vertex domain** , **graph domain** or **data space domain**. In the contrast, **Spectral domain** is also denoted as **feature space domain**.



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
| $n$ | $n = \|V\|$ the number of nodes |
| $m$ | $m = \|E\|$ the number of  edges |
| $d$ | the dimension of node feature |
| $h$ | the hidden dimension of node feature |
| $c$ | the dimension of edge feature |
| $L$ | Laplacian eigen map |
| $\mu_i, \lambda_i$ | $i$-th eigen vector and eigen value of Laplacian matrix |
| $\phi$ | $\phi = [ \mu_1, \cdots, \mu_n]$  Graph Fourier Inverse Transformation Operator |

## Global View

### Laplacian Eigen Map and Total Variation

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


Now, let's find some interesting properties of $L$. we simply suppose that the dimension of node feature is 1. Note when it is more than 1, we can conduct the following procedure  in a **predictor-wise**  manner to get the same perspective, thus $X = \left[\begin{matrix} x_1 & x_2 & \cdots & x_n\end{matrix}\right]^T \in R^n$ and $x_i$ is a scalar feature of node $i$. In addition, we suppose the graph is constructed from hard kernel (limitation of soft kernel graph) , thus $A \in \{ 0, 1\}^{n \times n}$.


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
&where \, e_{1i} \, e_{1j} \, e_{2i} \, e_{2j} \, e_{ni} \, e_{nj} \cdots \in E \\
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
- that decomposition of Laplacian Eigen Map $L\mu = \lambda\mu$ has trivial solutions when $\lambda = 0$ (I will show $\lambda \ge 0$ next paragraph) and the number of trivial solutions (the multiplicity of the smallest eigen value $0$ ) is just **the number of connected components** (Note there is one trivial solution at least , that is with the same value for all nodes), because we can assign a fixed value for every connected components and make every node in this component with this value. **Same or not same** value(s) for different connected components is dependent on the specific situation.
- that **spectral clustering** is a method for clustering ( dimensionality reduction and clustering), which just partitions the low-frequency base signals (regarding eigen vectors corresponding to **the second least** eigen value as reduced results).

Let's take one more step in a $n$-element **quadratic form** or **standard form**:


$$
\begin{align}
    X^TLX &= \left[\begin{matrix}
    x_1 & x_2 & \cdots & x_n
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

- that $L$ is a semi-positive definite matrix, and least eigen value $\lambda_1 = 0$ and I have proofed its trivial solutions ahead of time.

- that both $LX$ and $X^TLX$ represent the difference or variation between every node and its neighbors. we call the latter as **Total Variation** (denoted as **$TV(X)$**).

- Let's take one more step, supposing that $Y = \phi^TX = \left[\begin{matrix}
  y_1 & y_2 & \cdots & y_n
  \end{matrix}\right]^T$ is the embedding of **vertex signal/function** with the eigen vectors as the bases. $Y$ is also known as **eigen signal/function**.

  $$
  \begin{align}
  TV(X) &= X^TLX = X^T\phi \Lambda \phi^T X \\
  &= Y^T \Lambda Y \; (normalized \; form) \\
  &= \sum_{i = 1}^{n}{\lambda_iy_i^2}
  \end{align}
  $$
  
  

Now we have drawn a clear conclusion that $TV(X)$ is the squared sum of signal residuals between every node and its neighbors in **spatial domain** and is the squared sum of eigen-value-weighted eigen function/signal (coordinate representations or embeddings) in **spectral domain**.

Based on aforementioned analysis and given a topology structure of a graph, we can optimize the graph signals by $argmin \;TV(X)$ easily. Obviously, $TV(X)$ has the smallest value $0$ when vertex signal has the same direction with the eigen vector associated the least eigen value $\lambda_1 = 0$. More importantly, we have obtained this kind of eigen vectors by trivial solutions. In a word, $TV(X)$ has the smallest value $0$ if the vertex signal is made up of a fixed constant for all vertices of the whole graph or all vertices of current connected component. It is also intuitive to make this conclusion from definition of Laplacian Operator (**difference or variation**). On the contrary, $argmax \;TV(X)$ is a difficult optimization.

In addition, we can also observe the relations between **variation** and **spectral domain**. i.e., different eigen values have different contributions to the **variation/difference**. Specifically speaking, large eigen values contribute more than small eigen values. Therefore, if the embedding specific to the eigen vectors associated to large eigen values is **large**,  the graph signal tends to be **unsmooth, jittering and high-variance**. **On the contrary**, the graph signal is **smooth, mean and uniformly identical**. Please keep this observation in mind, I will formulate Graph Convolutional Networks from it.

So far, I have summarized the properties of $L$, different views and optimization of $TV(X)$ and obtained a intuitive but amazingly powerful inference that smoothness of graph signals is strongly related (or equivalent to) magnitude of embeddings of different eigen vectors. There is no doubt that optimization of $TV(X)$ is meaningless in the practical scenario, because It reassigns graph signals and ignores the original graph signals completely. Naturally, we need a kind of operation that **changes the smoothness of a given graph (i.e., low-pass, high-pass or band-pass)**, which is also known as **filtering** in digital signal processing (DSP) field.



### Graph Filtering

In concept , **graph filtering** is a 2-step procedure that consists of **graph-spectrum transformation (graph Fourier transformation) and modified reconstruction respectively**. In equation, it is represented as following:

- obtain embeddings using graph-spectrum transformation (map **from spatial domain to spectral domain**): 

  
  $$
  Y = \phi^TX = \left[\begin{matrix}
  y_1 \\
  y_2 \\
  \vdots \\
  y_n
  \end{matrix}\right]
  $$
  
- modify embeddings explicitly to smooth or unsmoooth the graph signals: 


$$
  \begin{align}
  \hat{Y} = h(\Lambda)Y = \left[\begin{matrix}
  h(\lambda_1) & 0 & \cdots & 0 \\
  0 & h(\lambda_2) & \cdots & 0 \\
  \vdots & \vdots & \ddots & \vdots \\
  0 & 0 & \cdots & h(\lambda_n)
  \end{matrix}\right] \left[\begin{matrix}
  y_1 \\
  y_2 \\
  \vdots \\
  y_n
  \end{matrix}\right] = \left[\begin{matrix}
  h(\lambda_1)y_1 \\
  h(\lambda_2)y_2 \\
  \vdots \\
  h(\lambda_n)y_n
  \end{matrix}\right]
  \end{align}
$$


- reconstruct the spatial graph signal (vertex function/signal) using modified embeddings (map **from spectral to spatial**):

  
  $$
  \hat{X} = \phi \hat{Y} = h(\lambda_1)y_1 \mu_1 + h(\lambda_2)y_2 \mu2 + \cdots + h(\lambda_n)y_n \mu_n
  $$

Aforementioned procedure is intuitive because smoothness is equivalent to embeddings of eigen vectors corresponding to large or small eigen values. Changing embeddings is just to adapt smoothness of graph signals. We can also regard this in a different view:



$$
\begin{align}
\hat{X} &= H X \\
& where \; H = \phi h(\Lambda) \phi^T
\end{align}
$$


In convolution theory, we conventionally call $H$ **filter matrix** and $h(\Lambda), h$ represent **response matrix** and **response function** respectively. Note **filter matrix** $H$ is also known as **Graph Shift Operator (GSO)**.

It is obvious 

- that graph filtering is just **a linear transformation** with filter matrix $H$.
- that different filter matrix $H$ will generate different transformed graph signals, such as
  - if $h$ is an identical mapping, $H = L$, $\hat{X} = LX$ is smoother than $X$ naturally (due to residuals). Furthermore, $L^kX$ will smoother than $L^{k-1}X$ (due to residuals of residuals), note $L^0X = X$. This kind of filter matrices is names **Chebyshev Base Filter Matrix** and I will discuss it later.
  - if  $h$ is an one mapping, $H = I_n$, $\hat{X} = X$ do nothing meaningful.
  - if $h$ is a zero mapping, $H = 0, \hat{X} = 0$ do nothing meaningful.
  - furthermore, although we have no explicit formations of response function $h$, we still claim that bot **adjacence matrix $A$** and **degree diagonal matrix D** are **GSOs**. I will regard $A$ as a GSO in later discussion of **GCN**.
- that properties of filter matrices / graph shift operators (GSOs) can be summarized as following:
  - linear transformation: $H(\alpha X + \beta Y) = \alpha HX + \beta HY$
  - commutative law (swappable): $H_1(H_2X) = (H_1H_2)X$
  - condition of inversible filtering: response function $h$ is inversible (we can replace original values with inversed values in the response matrix $h(\Lambda)$)

Because there are a great number of response functions $h$, we can choose different $h$ to construct different response matrices $h(\Lambda)$. However, please keep it in mind that the easy but power $h$ and $h(\Lambda)$ is identical mapping and laplacian eigen map $L$ respectively on account of the variation/difference of $L$.  Approximately, we can conduct **Taylor Expansion** w.r.t. $L$ ( or $\lambda_i$) to fit any filter matrix (GSO) $H$ ( or $h$ function), which is computed as:


$$
\begin{align}
h(\lambda_i) &= \sum_{k=0}^{K}{h_k \lambda_i^k} \\
h(\Lambda) &= \sum_{k=0}^{K}{h_k \Lambda^k} \\
H &= \phi h(\Lambda) \phi^T \\
&= \phi (\sum_{k=0}^{K}{h_k \Lambda^k}) \phi^T \\
&= \sum_{k=0}^{K}{h_k L^k} = \left[\begin{matrix}
\sum_{k=0}^{K}{h_k \lambda_1^k} & 0 & \cdots & 0 \\
0 & \sum_{k=0}^{K}{h_k \lambda_2^k} & \cdots & 0 \\
\vdots & \vdots & \ddots & \vdots \\
0 & 0 & \cdots & \sum_{k=0}^{K}{h_k \lambda_n^k}
\end{matrix}\right]\\
\end{align}
$$




This equation is trivial due to both **algebra view ($\phi \Lambda^k \phi^T = L^k $)** and **chain-based filtering transformation view ($L^{k+1}(X) = L(L^k(X))$)**. The aforementioned kind of filter matrix also known as **Chebyshev Network**. We can remember it by 3 ways ( i.e., Taylor Expansion of response function $h(\lambda_i)$, response matrix $h(\Lambda)$ and filter matrix (GSO)) $H = \sum_{k=0}^{K}{h_k L^k} $. 

Let's simplify this equation and explore the explicit relation between response function and filter matrix (note this relation is usually complicated and agnostic but here is a simple but generally power **Chebyshev Network**):




$$
\begin{align}
H &= \left[\begin{matrix}
\sum_{k=0}^{K}{h_k \lambda_1^k} & 0 & \cdots & 0 \\
0 & \sum_{k=0}^{K}{h_k \lambda_2^k} & \cdots & 0 \\
\vdots & \vdots & \ddots & \vdots \\
0 & 0 & \cdots & \sum_{k=0}^{K}{h_k \lambda_n^k}
\end{matrix}\right] \\
&= diag(\left[\begin{matrix}
\sum_{k=0}^{K}{h_k \lambda_1^k} & \sum_{k=0}^{K}{h_k \lambda_2^k} & \cdots & \sum_{k=0}^{K}{h_k \lambda_n^k}
\end{matrix}\right]^T) \\
&= diag(\left[\begin{matrix} 
1 & \lambda_1 & \lambda_1^2 & \cdots & \lambda_1^K \\
1 & \lambda_2 & \lambda_2^2 & \cdots & \lambda_2^K \\
\vdots & \vdots & \vdots & \ddots & \vdots \\
1 & \lambda_n & \lambda_n^2 & \cdots & \lambda_n^K \\
\end{matrix}\right] \left[\begin{matrix}
h_0 \\
h_1 \\
\vdots \\
h_K
\end{matrix}\right]) \\
&= diag(\Psi \pmb{h})
\end{align}
$$


where is $\Psi \in R^{n \times K} $ **Vandermonde Matrix** (not **$n$-order Vandermonde Determinant**), $ \pmb{h} $ is a coefficient vector, which represents **response function** $h$ naturally. 

when $n==K$, $ \pmb{h} = \Psi^{-1}diag^{-1}(H) $, but $K  << n$ is a common case.



So far I have obtained a relatively useful conclusion on graph filtering and Chebyshev Network best practice. However specifying response function $h$ or $\pmb{h}$ is not a wise idea, we wanna optimize it by a **differentiable and end-to-end** manner.



### Graph Convolutional Networks

## Local View

## Embedding View

## Spatial-Temporal View

