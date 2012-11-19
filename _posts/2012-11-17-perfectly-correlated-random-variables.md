---
layout: post
title: Perfectly Correlated Random Variables
category: probability
tags: [math]
---
{% include JB/setup %}

$\newcommand{\E}{\mathbb{E}}$Suppose $X$ and $Y$ have finite nonzero variances $\sigma_X^2$ and $\sigma_Y^2$. Define $C$ to be the correlation between $X$ and $Y$. If $\vert C \vert=1$, then $Y$ must be an affine linear function of $X$ (and vice versa) with probability one. In particular, $Y = mX + b$ where

<div>\begin{align*}
m = C \left(\frac{\sigma_Y}{\sigma_X}\right) \qquad \text{and} \qquad b = \E Y - m \E X
\end{align*}</div>


### Proof

Consider the variance of $Y-mX$.

<div>\begin{align*}
\text{Var}(Y-mX) &= \sigma_Y^2 + m^2\sigma_X^2 - 2m\text{Cov}(X,Y)\\
 &= \sigma_Y^2 + m^2\sigma_X^2 - 2m C \sigma_X \sigma_Y \qquad \text{(by the definition of correlation)}\\
 &= \sigma_Y^2 + (C \sigma_Y / \sigma_X)^2\sigma_X^2 - 2 (C \sigma_Y / \sigma_X) C \sigma_X \sigma_Y\\
 &= \sigma_Y^2 + C^2 \sigma_Y^2 - 2 C^2 \sigma_Y^2\\
 &= \sigma_Y^2 + \sigma_Y^2 - 2 \sigma_Y^2 \qquad \qquad \qquad \text{(because $C^2=1$)}\\
 &= 0
\end{align*}</div>

The variance of the random variable $Y-aX$ is zero.  A random variable with zero variance has to be a constant with probability one, so $Y-mX=b$ with probability one for some constant $b$. Taking expectations of both sides shows that $b$ must be $\E Y - m \E X$.

