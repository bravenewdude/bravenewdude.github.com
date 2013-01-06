---
layout: post
title: Estimating Stock Volatility
category: stochastic processes
tags: [finance]
---
{% include JB/setup %}


One of my earlier posts derives the [Black-Scholes formula](/stochastic processes/2012/12/28/the-black-scholes-model/) for stock prices. The main unknown variable in the formula is the stock's volatility. This post describes a couple methods for estimating this volatility, based largely on \textit{An Elementary Introduction to Mathematical Finance} by Sheldon Ross.

Any arbitrarily small interval of Brownian Motion is sufficient to estimate $\sigma$ exactly, because the interval can be subdivided into infinitely many subintervals, which provide infinitely many data points. With this knowledge, one might assume that it is trivial to find the volatility of a stock. Unfortunately, however, it's not that simple. While stock prices are modelled by GBM over everyday time frames, they do not have GBM properties over very short time intervals. The longer an interval is, the more it's overall behavior will resemble GBM. Another problem is that each stock's volatility may change gradually over time. More recent time intervals will provide better information regarding a stock's current volatility.

First, (Ross sections 8.5.1 - 8.5.2) consider the idealized case in which we have an interval of length $T$ with perfect GBM($\mu,\sigma$)-distributed data, given by $U(t)$. We can partition this interval into $n$ equal subintervals, each of which has length $l := T/n$. This allows us to define $n$ random variables, using notation from the Black-Scholes post,

<div>\begin{align*}
X_i &:= \log \left( \frac{U(i l)}{U((i-1)l)} \right)\\
 &= \log U(i l) - \log U((i-1)l)\\
 &= (b_0 + i l \mu + \sigma W(i l)) - (b_0 + (i-1) l \mu + \sigma W((i-1)l))\\
 &= l \mu + \sigma \left(W(i l) - W((i-1)l)\right)
\end{align*}</div>
each of which is normal with mean $l \mu$ and variance $l \sigma^2$. These random variables are independent because of the Markov property of Brownian Motion.

The random variable $Y := \sum(X_i - \bar{X})^2/(\sigma^2 l)$ has a $\chi^2_{n-1}$ distribution, so the standard unbiased estimator of $\sigma^2$ is $S^2/l$, where $S^2 := \sum(X_i - \bar{X})^2/(n-1)$. The mean-squared error of this estimator is equal to its variance
<div style='visibility: hidden; height: 0;'>$\newcommand{\V}{\text{Var}}$</div>

<div>\begin{align*}
\V \frac{\sum(X_i - \bar{X})^2}{l(n-1)} &= \frac{\sigma^4}{(n-1)^2} \V \frac{\sum(X_i - \bar{X})^2}{\sigma^2 l}\\
 &= \frac{\sigma^4}{(n-1)^2} \V Y\\
 &= \frac{\sigma^4}{(n-1)^2} 2(n-1) \qquad \text{because $\chi^2_r$ has variance $2r$}\\
 &= \frac{2 \sigma^4}{n-1}
\end{align*}</div>

Thus, if we had true GBM, then we could let $n$ be as large as we like by splitting our interval into smaller pieces, and thereby estimate squared volatility, and [therefore volatility](/inference/2012/12/29/an-unbiased-estimator-for-normal-standard-deviation), with arbitrary precision. The problem is, actual stock prices only behave like GBM when the time intervals are large enough.

To demonstrate this issue, I have performed a simulation. It generates a 6.5 hour interval of GBM (equal to one trading day on the stock market) refined to $2^4$ units of time (about a minute and a half each). This process has the "look" of GBM, as long as we do not zoom in too far. The squared volatility of the simulated GBM (with true volatility of 1) was estimated as described above, splitting our data into $n = 2^k$ data points for $k = 2,3,4,5,6$. This process was repeated 10000 times, and the squared error for each $n$ was computed in each trial. Note that the code below uses the `GenerateBM` [function](/stochastic processes/2012/12/28/the-black-scholes-model#Generate) defined in the Black-Scholes article.


    EstVol <- function(n, GBM) {
        # Estimate Volatility of GBM by splitting it into n units.
        GBM.y <- GBM[, 2]
        GBM.x <- GBM[, 1]
        m <- nrow(GBM)
        begin <- GBM.x[1]
        end <- GBM.x[m]
        l <- (end - begin)/n
        y <- GBM.y[1]
        if (n > 2) {
            j <- 1
            for (a in seq(begin + l, end - l, by = l)) {
                while (GBM.x[j + 1] <= a) j <- j + 1
                y <- c(y, GBM.y[j])
            }
        }
        y <- c(y, GBM.y[m])
        v <- log(y[-1]/y[-(n + 1)])
        est <- sum((v - mean(v))^2)/(n - 1)/l
        return(est)
    }
    
    set.seed(1)
    nsim <- 10000
    refinement <- 2^(2:6)
    errors <- matrix(NA, nsim, length(refinement))
    estimates <- matrix(NA, nsim, length(refinement))
    for (i in 1:nsim) {
        GBM <- GenerateBM(end = 6.5, mu = 0.01, log2n = 4, geometric = T)
        estimates[i, ] <- sapply(refinement, EstVol, GBM)
        errors[i, ] <- (estimates[i, ] - 1)^2
    }
    results <- cbind(refinement, apply(estimates, 2, mean), apply(errors, 2, mean))
    colnames(results) <- c("n", "avg volatility est", "avg sq error of est")
    
    results


    ##       n avg volatility est avg sq error of est
    ## [1,]  4              1.021              0.6818
    ## [2,]  8              1.015              0.2913
    ## [3,] 16              1.008              0.1347
    ## [4,] 32              1.008              0.1284
    ## [5,] 64              1.008              0.1270



The above table shows the average squared error for each value of $n$. We can see that the estimates tend to concentrate closer to the true value of 1 as $n$ increases to $2^4$, because up to that point we are incorporating more good data points. After that, increasing $n$ does not seem to reduce squared error very much. With too much refinement, the intervals are less like GBM. This demonstrates the trade-off between having a larger sample size and having more reliable data points.

Intervals on the order of *one trading day* seem to make the GBM assumption reasonable. It is probably not a good idea to refine any further because volatility systematically changes over the course of the day; prices change most rapidly near the opening and closing of each day. This doesn't fit the GBM assumption, which includes a fixed volatility.

Furthermore, we should not use all historical data for a stock, because its volatility changes as the company and investor psychology evolve over the years. Ross recommends looking only at the past year (252 trading days) to estimate a stock's current volatility.

The first method Ross proposes for estimating volatility uses only a stock's daily closing prices. Let us define the unit time interval to be one year, and partition a year of stock data into its 252 trading days. Thus, we are using $l = 1/252$. The 252 Normal$(\mu,\sigma^2/252)$ variables that we construct are $X\_i := \log (C\_i/C\_{i-1})$, where $C_k$ denotes the stock's closing price on trading day $k$, and $C_0$ is defined to be the closing price on the day before day 1. Then the standard procedure described above tells us to estimate $\sigma^2$ by $S^2/l = 252 S^2$. This estimator of $\sigma^2$ is unbiased, and our mean squared error is $2 \sigma^4 / (n-1) = 2 \sigma^4 / 251$. (Ross points out that $\mu$ is usually negligible compared to its standard deviation, so the estimator $(252/n)\sum X_i^2$ has roughly the same performance as $252 S^2$.)

Another method for estimating volatility involves using both the daily opening and closing stock prices (Ross 8.5.3). Let $O_i$ denote the opening price on day $i$. Now, recall that $\V X_i = \sigma^2/252$ and note that we can re-express $X_i$ as

<div>\begin{align*}
\log \left( \frac{C_i}{C_{i-1}} \right) &= \log \left( \frac{C_i}{O_i} \frac{O_i}{C_{i-1}} \right)\\
 &= \log \left( \frac{C_i}{O_i} \right) + \log \left( \frac{O_i}{C_{i-1}} \right)\\
 &= B_i + A_i
\end{align*}</div>
where $B_i := \log C_i/O_i$ and $A_i := \log O_i/C_{i-1}$. Therefore, assuming $A_i$ and $B_i$ are independent, $\sigma^2/252 = \V X_i = \V (B_i + A_i) = \V B_i + \V A_i$. Because $A_i$ and $B_i$ are normal with negligible means (compared to the standard deviations of the means), we can estimate their variances by $(1/n)\sum A_i^2$ and $(1/n)\sum B_i^2$, respectively. Therefore, we can estimate $\sigma^2$ by $(252/n)\sum (B_i^2 + A_i^2)$ (i.e. estimate $\sigma$ by $\sqrt{(252/n)\sum (B_i^2 + A_i^2)}$).

Ross claims that the second method of estimating $\sigma^2$ is better than the first method. In fact, this method is basically equivalent to splitting the initial GBM into twice as many equal intervals, which roughly doubles the precision of the $\sigma^2$ estimate, as long as the pieces still look like GBM at that resolution.

A third estimator for squared volatility uses the fact that $\sum X_i^2 / l \sigma^2$ has a noncentral chi-squared distribution with parameters $k = n$ and $\lambda = n l^2 \mu^2 / \sigma^2$. Therefore, the estimator $V := \sum X_i^2 / nl$ has mean $\sigma^2 + l \mu^2$ and variance $2 \sigma^4 / n + 4 l \mu^2 \sigma^2 / n$. It has a positive bias that is small when $\mu$ and/or $l$ are small relative to $\sigma$. This estimator has a lower mean-squared error than $S^2/l$ has when $2\sigma^4/(n-1) > 2\sigma^4/n + 4 l \mu^2 \sigma^2 / n + l^2 \mu^4$, which is satisfied when

<div>\begin{align*}
\left(\frac{\mu}{\sigma}\right)^2 \leq \frac{-2/(n-1) + \sqrt{2}\,\sqrt{3n^2 - 5n + 2}}{n(n-1)l}
\end{align*}</div>
The estimator $V$ tends to be preferable when $n$ and $l$ are both small. For example, in the case of my pre-circuit-breaker TAQ data (MAKE LINK), we have $n = 18$ dates and use $l = 5$ minutes, making the square root of the right-hand side of the inequality about 67.


    rightside <- function(n, l) {
        num <- -2/(n-1) + sqrt(2)*sqrt(3*n^2 - 5*n + 2)
        denom <- n*(n-1)*l
        return(num/denom)
    }
    
    sqrt(rightside(18, 1/(20*6.5*252)))


    ## [1] 66.99978



In practice, stock prices typically have comparable volatility and drift parameters at the one-year scale, so this inequality is almost certainly satisfied. To understand why this is, realize that the drift parameter is inconsequential in determining price fluctuations on a 5-minute time scale. So pretending $\mu=0$ introduces almost no bias but decreases variance.



