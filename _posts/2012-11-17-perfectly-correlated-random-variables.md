---
layout: post
title: Perfectly Correlated Random Variables
category: probability
tags: [math]
---
{% include JB/setup %}

Suppose $X$ and $Y$ have finite nonzero variances $\sigma_X^2$ and $\sigma_Y^2$. If $\vert\text{corr}(X, Y)\vert=1$, then there must exist nonzero constants $a$ and $b$ such that $Y = aX + b$ with probability one.

### Proof

$\vert\text{corr}(X,Y)\vert=1$ implies that one of the following must be true:

- Cov$(X,Y)=\sigma_X \sigma_Y$
- Cov$(X,Y)=-\sigma_X \sigma_Y$

Assume the first case is true (i.e. Cov$(X,Y)=\sigma_X \sigma_Y$). Then consider the variance of $Y-aX$.

<div>\begin{align*}
\text{Var}(Y-aX) &= \sigma_Y^2 - 2a\text{Cov}(X,Y) + a^2\sigma_X^2\\
 &= \sigma_Y^2 - 2a\sigma_X \sigma_Y + a^2\sigma_X^2 \qquad \text{by our assumption}\\
 &= (\sigma_Y - a\sigma_X)^2
\end{align*}</div>

which equals zero if $a=\sigma_Y/\sigma_X$.

Assuming the second case is true (i.e. Cov$(X,Y)=-\sigma_X \sigma_Y$), then $a=-\sigma_Y/\sigma_X$ makes Var$(Y-aX)=0$.

Therefore, whenever $\vert\text{corr}(X,Y)\vert=1$, there exists a constant $a$ such that the variance of the random variable $Y-aX$ is zero.  A random variable with zero variance has to be a constant with probability one, so $Y-aX=b$ (equivalently, $Y=aX+b$) with probability one for some constant $b$.

