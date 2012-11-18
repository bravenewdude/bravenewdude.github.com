---
layout: post
title: Perfectly Correlated Random Variables
category: probability
tags: [math]
---
{% include JB/setup %}

Suppose <span>$X$</span> and <span>$Y$</span> have finite nonzero variances <span markdown="0">$\sigma_X^2$</span> and <span>$\sigma_Y^2$</span>. If <span>$|\text{corr}(X, Y)| = 1$</span>, then there must exist nonzero constants <span>$a$</span> and <span>$b$</span> such that <span>$Y = aX + b$</span> with probability one.

### Proof

<span>$|\text{corr}(X,Y)|=1$</span> implies that one of the following must be true:

- Cov<span>$(X,Y)=\sigma_X \sigma_Y$</span> 
- Cov<span>$(X,Y)=-\sigma_X \sigma_Y$</span>

Assume the first case is true (i.e. Cov<span>$(X,Y)=\sigma_X \sigma_Y$</span>). Then consider the variance of <span>$Y-aX$</span>.
<div>\begin{align*}
\text{Var}(Y-aX) &= \sigma_Y^2 - 2a\text{Cov}(X,Y) + a^2\sigma_X^2\\
 &= \sigma_Y^2 - 2a\sigma_X \sigma_Y + a^2\sigma_X^2 \qquad \text{by our assumption}\\
 &= (\sigma_Y - a\sigma_X)^2
\end{align*}</div>
which equals zero if <span>$a=\sigma_Y/\sigma_X$</span>.

Assuming the second case is true (i.e. Cov<span>$(X,Y)=-\sigma_X \sigma_Y$</span>), then <span>$a=-\sigma_Y/\sigma_X$</span> makes Var<span>$(Y-aX)=0$</span>.

Therefore, whenever <span>$|\text{corr}(X,Y)|=1$</span>, there exists a constant <span>$a$</span> such that the variance of the random variable <span>$Y-aX$</span> is zero.  A random variable with zero variance has to be a constant with probability one, so <span>$Y-aX=b$</span> (equivalently, <span>$Y=aX+b$</span>) with probability one for some constant <span>$b$</span>.

