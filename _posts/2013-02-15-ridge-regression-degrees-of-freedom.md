---
layout: post
title: Ridge Regression Degrees of Freedom
category: machine learning
tags: [r]
---
{% include JB/setup %}


When running a [ridge regression](http://en.wikipedia.org/wiki/Ridge_regression), you need to choose a ridge constant $\lambda$. More likely, you want to try a set of $\lambda$ values, and decide among them by, for instance, [cross-validation](http://en.wikipedia.org/wiki/Cross-validation_%28statistics%29).

But what range of $\lambda$ values make sense for any given ridge regression? One answer is the set of $\lambda$ that correspond to the ridge regression's effective degrees of freedom (df) between 0 and the number of variables. If you could use a grid over that range of df, you would certainly cover the range of $\lambda$ values that make sense.

Calculating $\lambda$ [from df](http://stats.stackexchange.com/questions/8309/how-to-calculate-regularization-parameter-in-ridge-regression-given-degrees-of-f) can be accomplished numerically. The code below uses the [Newton Raphson](https://en.wikipedia.org/wiki/Newton%27s_method) algorithm to find a set of $\lambda$ values for your ridge regression, corresponding to a grid of df values.


<a id="DFtoLambda"></a>
{% highlight r %}
NewtonRaphson <- function(f, fprime, x = 1, ..., zero = 1e-05) {
    # Check provided arguments for derivative function
    mf <- match.call(expand.dots = FALSE)
    m <- match(c("fprime"), names(mf))
    # If a derivative function was not provided, create a numerical one
    if (is.na(m)) {
        fprime <- function(x, ...) {
            dx <- 0.001
            dy <- f(x + dx/2, ...) - f(x - dx/2, ...)
            return(dy/dx)
        }
    }
    # Iterate Newton Raphson algorithm until zero
    while (abs(f(x, ...)) > zero) {
        x <- x - f(x, ...)/fprime(x, ...)
    }
    return(x)
}

DFtoLambda <- function(X, r = 0.1) {
    # Returns a set of lambdas (in increasing order) corresponding to ridge
    # regression's effective degrees of freedom from 0 to the number of
    # columns of X with refinement r.
    X <- as.matrix(X)
    dsq <- eigen(t(X) %*% X, only.values = T)$values
    df <- seq(ncol(X), 0, by = -r)
    lam <- rep(NA, length(df))
    
    h <- function(lam, dsq, df) {
        return(sum(dsq/(dsq + lam)) - df)
    }
    
    hprime <- function(lam, dsq, df) {
        return(-sum(dsq/(dsq + lam)^2))
    }
    
    lam[1] <- 0
    for (i in 2:length(lam)) {
        # Use previous lambda as initial value
        lam[i] <- NewtonRaphson(h, hprime, x = lam[i - 1], dsq = dsq, df = df[i])
    }
    return(lam)
}
{% endhighlight %}


For example,


{% highlight r %}
DFtoLambda(USArrests)


##  [1] 0.000e+00 3.028e+01 6.577e+01 1.077e+02 1.576e+02 2.174e+02 2.898e+02
##  [8] 3.779e+02 4.859e+02 6.191e+02 7.842e+02 9.903e+02 1.249e+03 1.578e+03
## [15] 1.999e+03 2.545e+03 3.263e+03 4.224e+03 5.525e+03 7.301e+03 9.732e+03
## [22] 1.305e+04 1.753e+04 2.359e+04 3.183e+04 4.323e+04 5.952e+04 8.378e+04
## [29] 1.218e+05 1.842e+05 2.875e+05 4.509e+05 6.912e+05 1.028e+06 1.497e+06
## [36] 2.167e+06 3.184e+06 4.887e+06 8.305e+06 1.857e+07 3.365e+11
{% endhighlight %}



