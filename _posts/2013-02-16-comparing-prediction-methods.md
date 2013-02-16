---
layout: post
title: Comparing Prediction Methods
category: machine learning
tags: [r]
---
{% include JB/setup %}

Assume I have an data set of $n$ cases of $p$ predictor variables and one response variable. The relationships between the predictors and the response are all approximately linear. In the future, I will get new cases of the predictor variables, and I hope to predict their corresponing response variables as accurately as possible. I'd like to consider three popular types of prediction methods: subset selection, ridge regression, and lasso regression. Which one is likely to give me the best predictions of my future data?

One intuitive approach involves splitting my original data randomly into training and test groups, and seeing how well the methods predict the test data from the training data.

(For information on these and other prediction methods, I recommend *Elements of Statistical Learning* by Hastie, Tibshirani, and Friedman, which is an outstanding textbook. It also discusses estimation of prediction error.)


## compareMSPE

To choose a "best" prediction method, we need to decide what exactly we want to optimize. Let's assume that we want to minimize mean squared prediction error (MSPE).

I wrote a function `compareMSPE` to perform a sensible MSPE estimation process for any given set of prediction methods.

By default, `compareMSPE` compares subset selection, ridge regression, and lasso regression. To use `compareMSPE` for other methods, you will probably need to write your own helper functions, following the examples of `best.subset`, `best.ridge`, and `best.lasso`.

The default methods (subset, ridge, and lasso) all assume that the relationships between the response and the predictor variables are all approximately linear. The `pairs` function is a convenient way to judge whether this assumption is reasonable or not. In cases where a nonlinear relationship exists, often you can transform variables to get something close to linearity. 

Otherwise, you could code up functions for non-linear prediction methods, such as k-nearest neighbors regression, and pass these functions into `compareMSPE`.

The function requires a numerical response variable. The requirements on the predictor variables will depend on the mechanics of the prediction method functions that `compareMSPE` calls. For example, perhaps categorical variables will need to be coded by dummy variables, in some cases.

Below, you will notice several utility functions in the mix as well. The only one that `compareMSPE` really requires is `which.response`. However, its default prediction methods require the others. If you supply your own prediction methods, you will not need some of these other utility functions.


{% highlight r %}
which.response <- function(formula, data) {
    # Determines which column of data is the response variable in the formula
    mf <- match.call()
    m <- match(c("formula", "data"), names(mf))
    mf <- mf[c(1L, m)]
    mf[[1L]] <- as.name("model.frame")
    mf <- eval(mf, parent.frame())
    predictors <- labels(terms(mf))
    response <- which(!(names(data) %in% predictors), useNames = F)
    return(response)
}

ridge.predict <- function(formula, train, test, lam) {
    # Predict test response values using a ridge regression fit on the
    # training data for a range of lambda values.
    coef <- lm.ridge(formula, train, lam = lam)$coef
    r <- which.response(formula, train)
    test.x <- scale(test[, -r])
    return(mean(train[, r]) + test.x %*% coef)
}

crossval <- function(formula, data, K = min(10, nrow(data)), method, ...) {
    # Performs K-fold cross validation to estimate mean squared prediction
    # error for a range of parameter values
    n <- nrow(data)
    # permute rows into a random order
    data <- data[sample(1:n), ]
    # find out which column of data is the response var
    r <- which.response(formula, data)
    # assign rows to K groups
    group <- sort(rep(1:K, n/K + 1)[1:n])
    yhat <- foreach(i = 1:K, .combine = rbind) %do% {
        test <- data[group == i, ]
        train <- data[group != i, ]
        # apply prediction method to get estimates
        return(method(formula, train, test, ...))
    }
    MSPE <- apply(data[, r] - yhat, 2, function(x) mean(x^2))
    return(MSPE)
}

best.subset <- function(formula, data, new = NA) {
    # Uses the step function and the BIC criterion to choose a reduced model
    # and returns the coefficients for the full model (including zeros).
    l <- lm(formula, data)
    l <- step(l, trace = F, k = log(nrow(data)))
    coef <- rep(0, ncol(data))
    r <- which.response(formula, data)
    data <- data[, -r]
    m <- match(names(l$coef), names(data))
    if (is.na(m[1])) 
        m[1] <- 1
    coef[m] <- l$coef
    pred <- as.matrix(cbind(1, new)) %*% coef
    return(list(coef = coef, pred = pred))
}

best.ridge <- function(formula, data, new = NA) {
    # Returns coefficients of the best ridge regression, as determined by
    # 10-fold cross validation.
    cvridge <- crossval(formula, data, method = ridge.predict, lam = 0:200)
    lam <- (0:200)[which.min(cvridge)]
    l <- lm.ridge(formula, data, lam = lam)
    coef <- coef(l)
    pred <- as.matrix(cbind(1, new)) %*% coef
    return(list(coef = coef, pred = pred))
}

best.lasso <- function(formula, data, new = NA) {
    # Returns coefficients of the best lasso regression, as determined by
    # 10-fold cross validation.
    r <- which.response(formula, data)
    x <- as.matrix(data[, -r])
    y <- data[, r]
    cvlasso <- cv.lars(x, y, type = "lasso", plot.it = F)
    frac <- cvlasso$index[which.min(cvlasso$cv)]
    l <- lars(x, y, type = "lasso")
    coef <- predict.lars(l, type = "coefficients", s = frac, mode = "fraction")$coef
    intercept <- mean(y - x %*% coef)
    coef <- c(intercept, coef)
    pred <- as.matrix(cbind(1, new)) %*% coef
    return(list(coef = coef, pred = pred))
}

compareMSPE <- function(formula, data, methods = c(best.subset, best.ridge, 
    best.lasso), nsim = 20, verbose = F) {
    # Compare estimates of the mean squared prediction error for a list of
    # linear prediction methods.
    n <- nrow(data)
    r <- which.response(formula, data)
    results <- matrix(NA, nrow = nsim, ncol = length(methods))
    colnames(results) <- as.character(substitute(methods))[-1]
    for (i in 1:nsim) {
        # Split data into training and test sets
        s <- sample(1:n, floor(0.8 * n))
        test <- data[-s, ]
        train <- data[s, ]
        # Estimate MSPE for each method
        results[i, ] <- foreach(method = methods, .combine = c) %do% {
            yhat <- method(formula, train, test[, -r])$pred
            return(mean((test[, r] - yhat)^2))
        }
        if (verbose) 
            cat("Simulation", i, "complete.\n")
    }
    print(apply(results, 2, mean))
    return(results)
}
{% endhighlight %}


Finally, let's run `compareMSPE` on a data set to see which method performs best.


{% highlight r %}
library(MASS)  # required for best.ridge
library(lars)  # required for best.lasso
library(foreach)  # required for compareMSPE


results <- compareMSPE(Assault ~ ., USArrests)


## best.subset  best.ridge  best.lasso 
##       12930        2523        2532
{% endhighlight %}


Let's look at boxplots to get a feel for the distributions of the prediction errors for the various methods that were tried.

{% highlight r %}
par(mfrow = c(1, 2))
boxplot(results, main = "Avg Squared Prediction Errors")
# Now, zoom in on the better two methods
boxplot(results[, 2:3], main = "Avg Squared Prediction Errors")
{% endhighlight %}

{:.center}
![plot of chunk unnamed-chunk-6](/static/2013-02-16-comparing-prediction-methods/unnamed-chunk-6.png) 


For this data, ridge and lasso regression appear to be about equally good at predicting assault, and they are both much better than subset selection.


## Caution

Use this function in moderation! In general, don't use this function or other "all powerful" functions to do all the work for you. The most effective data analysis requires extensive thought and visualization. The more you manually play around with your data, the more likely you are to discover irregularities or to come up with insights. Then again, your time is valuable. Use automation with these trade-offs in mind.
