---
layout: post
title: Rejection Sampling Proof
category: inference
tags: [math]
---
{% include JB/setup %}
<div style='visibility: hidden; height: 0;'>$\renewcommand{\P}{\mathbb{P}} \newcommand{\E}{\mathbb{E}}$</div>

In the rejection sampling procedure, we wish to sample from some distribution with density $h$. To accomplish this, we first sample from some [dominating](http://en.wikipedia.org/wiki/Absolutely_continuous#Absolute_continuity_of_measures) distribution with density $g$. First find a value $M$ such that $M g(x) \geq h(x)$ for all $x$. Now, for each draw $Y$ from $g$, you also draw a standard uniform $U$. If $U \leq h(x)/(Mg(x))$, then set $X$ equal to $Y$. Otherwise draw from $g$ again and repeat the process.

It is easy to show that rejection sampling works. First, we'll need a couple of lemmas, both of which are quite interesting on their own.


## Lemma 1

If $A$ is a random variable whose support is a subset of $[0,1]$, and $U$ is a standard uniform random variable independent of $A$, then


<div>\begin{align*}
\P \{ U \leq A \} = \E A
\end{align*}</div>

### Proof

Let $f_A$ and $f_U$ be the densities of $A$ and $U$. Because they are independent, their joint density is equal to the product of the two marginal densities.


<div>\begin{align*}
\P \{ U \leq A \} &= \int_0^1 \int_0^a f_U (u) f_A (a) du \; da\\
 &= \int_0^1 \left[ \int_0^a f_U (u) du \right] f_A (a) da\\
 &= \int_0^1 \P \{ U \leq a \} f_A (a) da\\
 &= \int_0^1 a f_A (a) da\\
 &= \E A
\end{align*}</div>


## Lemma 2

The overall probability of accepting a draw is $1/M$.

### Proof


<div>\begin{align*}
\P \{ Y \; \text{is accepted} \} &= \P \{ U \leq h(Y)/(Mg(Y)) \}\\
 &= \E [h(Y)/(Mg(Y))] \qquad \qquad \qquad \text{(by Lemma 1)}\\
 &= \frac{1}{M} \int_{-\infty}^\infty \frac{h(y)}{g(y)} g(y) dy\\
 &= \frac{1}{M} \int_{-\infty}^\infty h(y) dy\\
 &= 1/M
\end{align*}</div>


## Rejection Sampling Theorem

The rejection sampling procedure described above produces draws from $X$ with density $h$.

### Proof


<div>\begin{align*}
\P \{X \in (x, x+\epsilon) \} &= \P \{ Y \in (x, x+\epsilon) \vert Y \; \text{is accepted} \}\\
 &= \frac{\P \{ Y \; \text{is accepted} \vert Y \in (x, x+\epsilon) \} \P \{ Y \in (x, x+\epsilon) \}}{\P \{ Y \; \text{is accepted} \}}\\
 &\approx \frac{ \P \{ U \leq h(Y)/(Mg(Y)) \vert Y \in (x, x+\epsilon) \} (\epsilon g(x))}{1/M}\\
 &\approx \epsilon M g(x) \P \{ U \leq h(x)/(Mg(x)) \}\\
 &= \epsilon M g(x) h(x)/(Mg(x))\\
 &= \epsilon h(x)
\end{align*}</div>

A routine limiting argument (letting $\epsilon \rightarrow 0$) shows that this implies that $X$ has density $h$.


