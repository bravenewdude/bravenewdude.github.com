---
layout: post
title: Comparing Affine Linear Gaussian DAGs and Gaussian Graphical Models
category: probability
tags: [math]
---
{% include JB/setup %}


Affine linear Gaussian DAGs (ALGDs) are a popular model in the causal discovery literature. Any multivariate Gaussian distribution on a vector $X$ of $k$ variables can be expressed as

<div>\begin{align*}
X &= m + L X + \Lambda \epsilon
\end{align*}</div>

where $m$ is a vector of intercepts, $L$ is a strictly lower triangular matrix of coefficients, $\Lambda$ is a diagonal matrix of residual standard deviations, and $\epsilon$ is a vector of independent standard normals. Any multivariate normal distribution actually has $k!$ different ALGD representations, one for each ordering of the variables (though many of these could produce the same DAG). If we have data that we believe to come from a simple causal mechanism in which each variable only depends on a few of its predecessors, then we might expect that the data would be well-approximated by a distribution that has sparse ALGD representations (sparse in the coefficient matrix).

The common way of representing a multivariate Gaussian distribution is via a mean vector and covariance matrix. It is easy to envision a causal mechanism of the affine linear Gaussian type that has a very sparse ALGD representation but a fully non-zero covariance matrix, for instance if each of $X_2, \ldots, X_k$ only depended on the preceding variable. In general, the ALGD representations can capture the sparsity of sparse causal mechanisms.

The same reasoning does not hold for the inverse covariance matrix, however. In many cases, it will retain much of the best ALGD's sparsity when the distribution corresponds to a sparse causal mechanism. A zero in the $(i,j)$ position of the inverse covariance matrix implies that variables $i$ and $j$ are independent conditional on all of the other variables. Revisiting the Markov-chain-like example, we see that the inverse covariance would be just as sparse as the sparsest ALGD.

The Gaussian graphical model (GGM) representation of a Gaussian distribution is parametrized by the elements of the inverse covariance matrix. The non-zero elements of the inverse covariance can be portrayed as undirected edges in a graph.

Given any DAG $G$, the *moralized graph* for $G$ is constructed by creating edges between any two nodes that share a child then removing the orientations of all edges. If a DAG $G$ corresponds to an ALDG representation of a distribution P, then the GGM for P is the moralized graph of $G$.

For a Gaussian distribution with covariance $\Sigma$, let us use the term *moral matrix* for the $k$ by $k$ matrix $M$ that agrees with $\Sigma^{-1}$ below the diagonal (strictly below) and is filled with zeros elsewhere. Note that $\Sigma^{-1}$ is equal to the sum of $M$ and $M'$ and a diagonal matrix. Then the non-zeros in $M$ correspond exactly to the edges of the GGM of our distribution.

Intuitively, ALGD sparsity captures our notion of causal complexity. Because it often achieves similar sparsity, we could say that GGM sparsity approximately  measures causal complexity also. However, the next result shows that a DAG representation \emph{can be} much sparser than an inverse covariance representation.

### Proposition 1 *(Comparing DAG Sparsity to GGM Sparsity)*

Given any $k$-dimensional multivariate Gaussian distribution, let $L$ be any sparsest ALGD coefficient matrix and $M$ be the moral matrix. Then
<div>\begin{align*}
\|M\|_0 - \|L\|_0 \leq \frac{(k-1)(k-2)}{2}
\end{align*}</div>
and there exist distributions achieving this bound.

### Proof

This can be shown by strong induction. The base case of $k=1$ is trivial. In that case, no edges are possible in either a DAG or GGM representation. Therefore, $\|M\|_0 = \|L\|_0 = 0$ and so $\|M\|_0 - \|L\|_0 \leq \frac{(1-1)(1-2)}{2} = 0$ as required.

Before proving the inductive step, we describe how to generate a distribution that achieves this upper bound. First, variables $X\_1, \ldots, X\_{k-1}$ are generated independently of each other. Then, $X_k$ is any linear combination (with strictly non-zero coefficients) of the previous $k-1$ variables plus Gaussian noise. Then each pair of variables are dependent conditioned on all the others. Thus the GGM is a complete graph and $\|M\|_0 = \binom{k}{2} = \frac{k(k-1)}{2}$. The sparsest DAG has edges from each of $X\_1, \ldots, X\_{k-1}$ to $X_k$ giving $\|L\|_0 = k-1$. This distribution achieves the upper bound with $\|M\|_0 - \|L\|_0 \leq \frac{(k-1)(k-2)}{2}$.

Now, we can prove the inductive step. First, notice that $\|M\|_0$ cannot exceed $\binom{k}{2}$ because it corresponds to the complete GGM. Therefore, any distribution with $\|L\|_0 \geq k-1$ cannot possibly make $\|M\|_0 - \|L\|_0$ larger than our proposed upper bound. Finally, consider the case that $\|L\|_0 < k-1$. Then there is a DAG representation of the distribution that does not have enough edges to span the full graph, leaving at least one isolated node. Each isolated node must represent a variable that is independent of all the others. Likewise, it will have no edges in the GGM. Finding the difference $\|M\|_0 - \|L\|_0$ reduces to the problem of finding this difference on a subset of variables with cardinality strictly less than $k$ (by ignoring the isolated nodes). If we adopt the inductive hypothesis that our upper bound statement holds for problems of size $1, \ldots, k-1$, then that bound must hold for our subproblem. That bound is less than $\frac{(k-1)(k-2)}{2}$, which completes the proof. $\square$

