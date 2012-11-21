---
layout: post
title: Sampling from an Arbitrary Density
category: 
tags: [r]
---
{% include JB/setup %}
<div class="hidden">$\newcommand{\I}{\mathbb{1}}$</div>

One way to sample from a known probability density function (pdf) is to use [inverse transform sampling](http://en.wikipedia.org/wiki/Inverse_transform_sampling). First, you integrate the pdf to get the cumulative distribution function (cdf). Next, you find the inverse of the cdf. Finally, apply this inverse cdf to each number in a sample of Uniform(0,1) observations.


## Example

Suppose we wish to sample from the pdf


<div>\begin{align*}
f(x) &= \frac{2m^2}{(1-m^2)x^3} \I\{x \in [m,1]\}
\end{align*}</div>

The cdf is


<div>\begin{align*}
F(x) &= \int_{-infty}^x f(t) dt\\
 &= \left\{
     \begin{array}{ll}
       0 & , x < m\\
       \frac{1}{1-m^2} - \frac{m^2}{(1-m^2)x^2} & , x \in [m,1]\\
       1 & , x > 1
     \end{array}
    \right.
\end{align*}</div>

For $u \in [0,1]$, its inverse is


<div>\begin{align*}
F^{-1}(u) &= \sqrt{\frac{m^2}{1-(1-m^2)u}}
\end{align*}</div>

Finally, we can apply the technique. Below, I generate 100 standard uniform observations and transform each using the inverse cdf.


    invcdf <- function(u, m = 0.5) {
        return(m^2/(1 - (1 - m^2) * u))
    }
    
    sample1 <- sapply(runif(100), invcdf)
    plot(density(sample1), main = "Sample Density using invcdf Function")


{:.center}
![plot of chunk unnamed-chunk-1](/static/2012-11-20-sampling-from-an-arbitrary-density/unnamed-chunk-1.png) 



## R Function for an Arbitrary pdf of One Variable

The R code below uses some of R's built-in numerical methods to accomplish the inverse transform sampling technique for any arbitrary pdf that it is given. I'm sure there are some ugly pdfs for which this function wouldn't work, but it works fine for typical densities.


    endsign <- function(f, sign = 1) {
        b <- sign
        while (sign * f(b) < 0) b <- 10 * b
        return(b)
    }
    
    samplepdf <- function(n, pdf, lower = -Inf, upper = Inf) {
        vpdf <- function(v) sapply(v, pdf)  # vectorize
        cdf <- function(x) integrate(vpdf, lower, x)$value
        invcdf <- function(u) {
            subcdf <- function(t) cdf(t) - u
            if (lower == -Inf) 
                lower <- endsign(subcdf, -1)
            if (upper == Inf) 
                upper <- endsign(subcdf)
            return(uniroot(subcdf, c(lower, upper))$root)
        }
        sapply(runif(n), invcdf)
    }



We can use `samplepdf` to sample from $f$, which we defined earlier.


    mypdf <- function(t, m = 0.5) {
        if (t >= m & t <= 1) 
            return(2 * m^2/(1 - m^2)/t^3)
        return(0)
    }
    
    sample2 <- samplepdf(100, mypdf)
    plot(density(sample2), main = "Sample Density using samplepdf Function")


{:.center}
![plot of chunk unnamed-chunk-3](/static/2012-11-20-sampling-from-an-arbitrary-density/unnamed-chunk-3.png) 



## Computational Speed of the Algorithm

While `samplepdf` is convenient, it is also computationally slow. The algorithm is $O(n)$, so it is easy to estimate the time required for a large sample size. Simply fit a linear model to the times required for a set of small sample sizes, as shown below.


    ntime <- function(n, f) {
        return(system.time(f(n))[3])
    }
    
    samplepdf.mypdf <- function(n) {
        return(samplepdf(n, mypdf))
    }
    
    n <- 100 * 1:10
    times <- sapply(n, ntime, f = samplepdf.mypdf)
    fit <- lm(times ~ n)$coeff
    print(paste("The number of seconds required to sample from this density is about", 
        fit[1], "+", fit[2], "* n."))


    ## [1] "The number of seconds required to sample from this density is about 3.01313333333334 + 0.143591212121212 * n."


    plot(n, times, ylab = "seconds", xlab = "n", main = "Time Required", col = 3)
    abline(fit, col = 2)


{:.center}
![plot of chunk unnamed-chunk-4](/static/2012-11-20-sampling-from-an-arbitrary-density/unnamed-chunk-4.png) 


At the beginning of this document, we found the inverse cdf analytically. The R function that used this analytical result can generate a sample of size 1000 in the blink of an eye, while the `samplepdf` function took about two and a half minutes on my slow netbook. However, if you give the lower and upper endpoints as in `samplepdf(1000, mypdf, .5, 1)`, the function only takes about ten seconds to run.

