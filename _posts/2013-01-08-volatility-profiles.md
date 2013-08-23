---
layout: post
title: Volatility Profiles
category: stochastic processes
tags: [finance]
---
{% include JB/setup %}


## Constant Volatility over Small Intervals

Stock prices are often modeled by Geometric Brownian Motion. Each stock is assumed to have a volatility parameter that is roughly stable over time frames on the order of, say, a year. In practice, stock prices tend to change much more rapidly at the beginning and end of each trading day than they do in the middle. To analyze intra-day volatilities, we need to use a more general diffusion model that allows the volatility to depend on $t$. I will refer to a stock's daily volatility pattern as its "volatility profile."

In theory, one could compute volatility estimates over arbitrarily small time intervals. However, the more you zoom in, the less "GBM-like" stock prices are. For our analysis, we broke each trading day ($T=6.5$ hours) into seventy-eight five-minute ($l=5$ minutes) intervals. On day $i$, represent a stock price in terms of a standard Brownian Motion $W_i$ by

<div>\begin{align*}
U_i (t) = \exp \left(b_{i,0} + t \mu + \sigma(t) W_i(t) \right)
\end{align*}</div>
We define $n=T/l=78$ random variables, one for each interval:

<div>\begin{align*}
X_{i,j} &:= \log \left( \frac{U_i(j l)}{U_i((j-1)l)} \right)\\
 &= \log U_i(j l) - \log U_i((j-1)l)\\
 &= (b_{i,0} + j l \mu + \sigma(j l) W_i(j l)) - (b_{i,0} + (j-1) l \mu + \sigma((j-1)l) W_i((j-1)l))\\
 &\approx l \mu + \sigma(j l - l/2) \left(W_i(j l) - W_i((j-1)l)\right) \qquad \text{by assumption explained below}
\end{align*}</div>
As a first attempt, we are assuming $\sigma(j l) \approx \sigma((j-1)l) \approx \sigma(j l - l/2)$, that is, volatility is nearly constant over small intervals. For a given stock, each $X_{i,j}$ is normal with mean $l \mu$ and variance approximately $l$ times $\sigma^2(j l - l/2)$ (henceforth abbreviated to $\sigma^2$), and they are independent of each other.

Assume we have data for $m$ trading days. The random variable

<div>\begin{align*}
Y_j := \sum_{i=1}^m(X_{i,j} - \bar{X}_j)^2/(l \sigma^2)
\end{align*}</div>
has an approximately $\chi^2\_{m-1}$ distribution, so the standard unbiased estimator of $\sigma^2$ is $S^2/l$, where $S^2 := \sum(X_{i,j} - \bar{X}\_j)^2/(m-1)$. The mean-squared error of this estimator is equal to its variance

<div style='visibility: hidden; height: 0;'>$\newcommand{\V}{\text{Var}}$</div>

<div>\begin{align*}
\V \frac{\sum(X_{i,j} - \bar{X}_j)^2}{l(m-1)} &\approx \frac{\sigma^4}{(m-1)^2} \V \frac{\sum(X_{i,j} - \bar{X}_j)^2}{l \sigma^2}\\
 &= \frac{\sigma^4}{(m-1)^2} \V Y\\
 &= \frac{\sigma^4}{(m-1)^2} 2(m-1) \qquad \text{because $\chi^2_r$ has variance $2r$}\\
 &= \frac{2 \sigma^4}{m-1}
\end{align*}</div>

Refer to earlier posts for an [analysis of this estimator, along with a comparison to other estimators](/stochastic processes/2012/12/30/estimating-stock-volatility/). Yet another post describes how to [estimate volatility](/inference/2012/12/29/an-unbiased-estimator-for-normal-standard-deviation/) rather than squared volatility.

Unfortunately, after glancing at some plots, the assumption of nearly constant volatility over five-minute intervals seems entirely untenable. Often, a volatility seems to change by a large proportion over the course of five minutes. Typical plots showing this can be found in the [another article](/tutorial/2013/01/14/single-stock-circuit-breakers/).


## Linearly Changing Volatility over Small Intervals

After realizing this problem, I devised a more plausible assumption: the change in volatility over any five-minute interval is approximately linear. Let $V(t)$ be a process whose natural logarithm is a diffusion with linearly changing $\sigma(t)$. That is,

<div>\begin{align*}
\log V(t) &= v_0 + \mu t + \int_0^t \sigma(\tau) d W(\tau)\\
 &= v_0 + \mu t + \int_0^t (\sigma_0 + m\tau) d W(\tau)
\end{align*}</div>

Let us zero in on the complicated part of this expression by defining $R(t) := \int\_0^t (\sigma_0 + m\tau) d W(\tau)$. At any given time $t$, the diffusion $R(t)$ is locally approximated by a Brownian Motion with volatility $\sigma\_0 + mt$. In other words, $R(t+\delta) - R(t)$ should approach a $N(0, (\sigma\_0 + mt)^2 \delta)$ distribution as $\delta \rightarrow 0$.

By transforming the time axis of a standard Brownian Motion in just the right way, we can produce a much more familiar-looking process with this same behavior. We need to find a transformation $D$ such that $W(D(t))$ has a "volatility" of $\sigma\_0 + mt$ at any time $t$. A change in time of $\delta$ from time $t$ must produce a change in $D$ by $(\sigma_0 + mt)^2 \delta$. That is, $D$ satisfies $D(t+\delta) = D(t) + (\sigma\_0 + mt)^2 \delta$ in the limit. Rearranging, and taking an anti-derivative, we find that a solution is $D(t) = \sigma\_0^2 t + \sigma\_0 m t^2 + m^2 t^3/3$. So our final result is

<div>\begin{align*}
W(\sigma_0^2 t + \sigma_0 m t^2 + m^2 t^3/3)
\end{align*}</div>

This diffusion has the same infinitesimal behavior as $R(t)$ at all times, so they are identical processes. The following simulations support this result. We will plot a diffusion on (0,1) with $\sigma(t) = 1-t$ (i.e. linear with $\sigma_0=1$ and $m=-1$). First, we generate the desired diffusion sequentially.

{% highlight r %}
LinearVolDiffusion <- function(end = 1, initial = 0, mu = 0, sigma0 = 1, m = 0, 
n = 1000) {
delta <- 1/n
x <- delta * (0:n)
y <- c(initial, rep(NA, n))
for (i in 1:n) {
    t <- x[i] + delta/2
    y[i + 1] <- y[i] + mu * delta + rnorm(1, sd = (sigma0 + m * t) * sqrt(delta))
}
return(cbind(x, y))
}

plot(c(0, 1), c(-1.5, 1.5), type = "n", main = "Sigma Changes by Step", xlab = "Time", 
 ylab = "Diffusion")
for (i in 1:4) {
lines(LinearVolDiffusion(m = -1), col = i + 1)
}
{% endhighlight %}

{:.center}
![plot of chunk unnamed-chunk-1](/static/2013-01-08-volatility-profiles/unnamed-chunk-1.png) 


Next, we generate the same diffusion using standard Brownian Motion and the $D$ transformation discovered earlier. Note that the `GenerateBM` function invoked below [can be found in an earlier post](/stochastic processes/2012/12/28/the-black-scholes-model#Generate).

{% highlight r %}
right <- function(f, b = 10) {
while (f(b) < 0) b <- 10 * b
    return(b)
}

D <- function(t, sigma0 = 1, m = -1) {
    return(sigma0^2 * t + sigma0 * m * t^2 + m^2 * t^3/3)
}

Dinv <- function(x, ...) {
    sub <- function(t) D(t, ...) - x
    return(uniroot(sub, lower = 0, upper = right(sub))$root)
}

plot(c(0, 1), c(-1.5, 1.5), type = "n", main = "SBM with Transformed Time Axis", 
    xlab = "Time", ylab = "Diffusion")
for (i in 1:4) {
    b <- GenerateBM(end = D(1))
    lines(sapply(b[, 1], Dinv), b[, 2], col = i + 1)
}
{% endhighlight %}

{:.center}
![plot of chunk unnamed-chunk-2](/static/2013-01-08-volatility-profiles/unnamed-chunk-2.png) 


The resemblance between the two plots gives us some reassurance that our results are correct.

Now consider the distribution of $\log V(l)/V(0)$.

<div>\begin{align*}
\log V(l)/V(0) &= \log V(l) - \log V(0)\\
 &= [v_0 + \mu l + W(\sigma_0^2 l + \sigma_0 m l^2 + m^2 l^3/3)] - [v_0]\\
 &= \mu l + W(l [(\sigma_0 + ml/2)^2 + (m l)^2/12])\\
 &\approx \mu l + W(l (\sigma_0 + ml/2)^2) \qquad \qquad \text{if $ml$ is small}
\end{align*}</div>

It is normally distributed with variance approximately $l (\sigma\_0 + ml/2)^2$, assuming the product of $m$ and $l$, which is the change in $\sigma$ over the interval, is much smaller than one. Plots indicate that this assumption holds up quite well overall, though not perfectly; we could always use a smaller interval if necessary to make the assumption more plausible. Therefore, it is easy to see that estimating the variance from a sample of random variables with this distribution allows us to estimate $(\sigma\_0 + ml/2)^2$, the squared volatility at the midpoint of the interval $(0,l)$. Likewise, it can be shown that the same process provides an estimate for the squared volatilities at the midpoints of the $n$ consecutive intervals of length $l$ from $(0,T)$.

The upshot is, we can still use the same estimator derived earlier to estimate the same quantities (midpoint $\sigma^2$ values), but now we have justified ourselves with a more believable model.

