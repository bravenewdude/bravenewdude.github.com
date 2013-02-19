---
layout: post
title: Minimax Set Example
category: decision theory
tags: [math]
---
{% include JB/setup %}

<div style='visibility: hidden; height: 0;'>$\newcommand{\E}{\mathbb{E}}$</div>
<div style='visibility: hidden; height: 0;'>$\newcommand{\I}{\mathbb{I}}$</div>
<div style='visibility: hidden; height: 0;'>$\renewcommand{\P}{\mathbb{P}}$</div>
<div style='visibility: hidden; height: 0;'>$\newcommand{\R}{\mathbb{R}}$</div>


This question is adapted from a past qualifying exam question. Let $\lambda$ denote [Lebesgue measure](https://en.wikipedia.org/wiki/Lebesgue_measure).

> Consider the choice of a set estimator $C_X$ for a parameter $\theta \in \R$ based on one observation $X$ from the $N (\theta, 1)$ distribution, using the loss function $L(\theta, C) = \frac{1}{4}\lambda C - \I ${$\theta \in C\$}. Show that the set $C\_X = [X − c\_0, X + c\_0]$ is [minimax](https://en.wikipedia.org/wiki/Minimax) for a suitably chosen constant $c\_0$, and find this $c\_0$.

### Solution

We will show that $C_X$ is minimax by showing that it is an equalizer decision rule and that it is extended Bayes. By Theorem 2.2 of *Essentials of Statistical Inference* by Young and Smith, these two conditions are sufficient for minimaxity.

Sometimes, minimaxity can be shown directly. More often, invoking this theorem is the right approach approach to take.

#### Equalizer

Let $\Phi$ denote the standard normal cdf.


<div>\begin{align*}
R(\theta, C_X) &= \E L(\theta, C_X)\\
 &= \E [ \tfrac{1}{4}\lambda C_X - \I \{\theta \in C_X\} ]\\
 &= \tfrac{1}{4}\lambda C_X - \P \{\theta \in C_X\}\\
 &= \tfrac{1}{4}\lambda [X-c_0, X+c_0] - \P \{\theta \in [X-c_0, X+c_0]\}\\
 &= \tfrac{1}{4}(2c_0) - \P \{X-c_0 \leq \theta \leq X+c_0\}\\
 &= c_0/2 - \P \{-c_0 \leq \theta - X \leq c_0\}\\
 &= c_0/2 - [1 - 2\Phi(-c_0)]
\end{align*}</div>

Given any $c_0$, the risk of the decision rule is the same for all $\theta$.

#### Extended Bayes

Consider a $N(0, \tau^2)$ prior distribution on $\theta$. Then the posterior distribution is


<div>\begin{align*}
\pi(\theta \vert X) &\varpropto f(X; \theta) \pi(\theta)\\
 &\varpropto \exp \left( \frac{-(X-\theta)^2}{2} \right) \exp \left( \frac{-\theta}{2\tau^2} \right)\\
 &\varpropto \exp \left( \frac{-(\tau^2+1) \theta^2 + 2X\tau^2 \theta}{2\tau^2} \right) &\text{then complete the square}\\
 &\varpropto \exp \left( \frac{-(\theta - tX)^2}{2t} \right) &\text{defining $t := \tau^2/(\tau^2+1)$}
\end{align*}</div>

which is $N(tX, t)$.

To minimize the [Bayes risk](https://en.wikipedia.org/wiki/Bayes_risk#Definition), it is sufficient to minimize the expected posterior loss.


<div>\begin{align*}
\inf_C \E [ L(\theta, C) \vert X] &= \inf_C \int (\tfrac{1}{4}\lambda C - \I \{\theta \in C\}) \pi(\theta \vert x) d\theta\\
 &= \inf_C (\tfrac{1}{4}\lambda C - \int \I \{\theta \in C\}) \pi(\theta \vert x) d\theta)\\
 &= \inf_C (\tfrac{1}{4}\lambda C - \P_{\theta \vert X} C)\\
 &= \inf_C (\tfrac{1}{4}\lambda C - \P_{\theta \vert X} [tX - .5 \lambda C, tX + .5 \lambda C]) \qquad \qquad \dagger\\
 &= \inf_C \left(\tfrac{1}{4}\lambda C - \P_{\theta \vert X} \left\{-\frac{1}{2\sqrt{t}} \lambda C \leq \frac{\theta - tX}{\sqrt{t}} \leq \frac{1}{2\sqrt{t}} \lambda C\right\}\right)\\
 &= \inf_C \left[\tfrac{1}{4}\lambda C - \left(1 - 2\Phi\left(-\frac{1}{2\sqrt{t}}\lambda C\right)\right)\right]
\end{align*}</div>

$\dagger$ For any fixed $\lambda C$, the (normal) posterior measure of $C$ is obviously maximal if $C$ is centered at the posterior mean.

By taking the derivative of the final expression with respect to $\lambda C$, we find that it is minimized at $\lambda C = 2 \sqrt{t} \phi^{-1} (1/4)$, where $\phi^{-1}$ is the inverse of the standard normal pdf restricted to $[0, \infty)$. By plugging in the interval with this width, centered at $tX$, we find the infimum expected posterior loss to be


<div>\begin{align*}
\frac{\sqrt{t}}{2} \phi^{-1} (1/4) - \left( 1 - 2\Phi ( -\phi^{-1}(1/4) ) \right)
\end{align*}</div>

Because the expression is continuous in $t$, and because $t \rightarrow 1$ as $\tau^2 \rightarrow \infty$, the expression converges (monotonically increasing) to


<div>\begin{align*}
L = \tfrac{1}{2}\phi^{-1}(1/4) + 2 \Phi (\phi^{-1}(1/4)) - 1
\end{align*}</div>

In other words, given any $\epsilon > 0$ there exists a $\tau\_1$ such that when $\tau > \tau\_1$, the infimum expected posterior loss is within $\epsilon$ of this limit. Likewise, the expectation of this infimum expected posterior loss, taken over any marginal distribution of $X$ (including the "true" one corresponding to any $\tau$) must be in $[L-\epsilon, L]$. Therefore, for any $\tau > \tau_1$, the infimum Bayes risk is in $[L-\epsilon, L]$.

Now, consider the interval $C\_X$ with $c\_0 = \phi^{-1}(1/4)$ (which is approximately 1). Because it is an equalizer, its Bayes risk equal to its constant risk. Looking back at this constant risk found in the previous part, and plugging in our $c\_0$, we find that $C\_X$ has Bayes risk equal to $L$.

If $\tau > \tau_1$, then the infimum Bayes risk is within $\epsilon$ of $L$, which is the Bayes risk of $C_X$. Therefore, $C_X = [X - \phi^{-1}(1/4), X + \phi^{-1}(1/4)]$ is $\epsilon$-Bayes, and thus minimax.

