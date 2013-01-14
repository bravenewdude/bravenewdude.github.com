---
layout: post
title: Single-Stock Circuit Breakers
category: tutorial
tags: [finance]
---
{% include JB/setup %}


On May 6, 2010, the U.S. stock market experienced what has come to be called the "[flash crash](http://en.wikipedia.org/wiki/May_6,_2010_flash_crash)," in which the prices of various stocks fluctuated violently. Overall, the market rapidly plunged about nine percent and recovered minutes later. In respone, the SEC and stock exchanges devised the Single-Stock Circuit Breaker (SSCB) rules, which were implemented gradually. According to [Traders Magazine](http://www.tradersmagazine.com/news/single-stock-circuit-breakers-sec-flash-crash-trading-106018-1.html),

> The single-stock circuit breakers will pause trading in any component stock of the  Russell 1000 or S\&P 500 Index in the event that the price of that stock has moved 10 percent or more in the preceding five minutes. The pause generally will last five minutes, and is intended to give the markets a hiatus to attract trading interest at the last price, as well as to give traders time to think rationally.

It is important to note that the rules do not apply to the first fifteen minutes and the last fifteen minutes of each trading day.

Under the direction of [Dr. Michael Kane](http://publichealth.yale.edu/people/michael_kane.profile), I am analyzing recent stock data. The aim of our research is to understand to effects of the SSCB rules. In particular, do the rules have any effect on the daily [volatility profiles](/stochastic processes/2013/01/08/volatility-profiles/) of stocks?


## Data Analysis

In my previous post, I describe how I [processed the TAQ data and acquired market cap data](/tutorial/2013/01/10/processing-taq-data). Now, for each stock I have a series of volatility estimates throughout the trading day over two distinct time periods: one period from 2010 before the SSCB policy and another period from 2011 after the SSCB rules were enacted.

The SSCB rules only apply to a subset of stocks: any stock in the [Russell 1000 Index](http://en.wikipedia.org/wiki/Russell_1000_Index) or the [S&P 500](http://en.wikipedia.org/wiki/S%26P_500). Unfortunately for our analysis, these stocks are systematically different from the average. They tend to be stocks of very large companies. This is obviously not ideal, but I will do my best to control for the systematic differences.

At this point, we have squared volatility profile estimates for 1695 stocks. 695 of these are in the SSCB group, while the remaining 1000 comprise the non-SSCB group.

I started by working with the data from 2010. During this period, the SSCB rules were not in place yet. This step establishes a baseline. Did the two groups of stocks have similar volatility profiles before the rules were enacted?

Here is a typical example of a stock's estimated squared volatility profile.


    vols <- read.csv("2010.csv", row.names = 1)
    t <- 2.5 + 5 * (0:77)
    
    set.seed(6)
    i <- runif(1, 1, nrow(vols))
    plot(t, vols[i, ], main = rownames(vols)[i], xlab = "Minutes into Day",
         ylab = "Squared Volatility")


{:.center}
![plot of chunk unnamed-chunk-1](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-1.png) 



### Spread Versus Level

It is clear from browsing more of these plots that the first and last points of the day tend to be the highest. It also seems to have the most variability. In fact, the following plot confirms that a strong relationship exists between spread and level. Times of the day with higher volatilities also have higher variation in their volatilities.


    means <- apply(vols, 2, mean)
    sds <- apply(vols, 2, sd)
    plot(means, sds, xlab = "Level", ylab = "Spread", main = "Original Data")


{:.center}
![plot of chunk unnamed-chunk-2](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-2.png) 


Generally, analyses go more smoothly if this relationship can be transformed away. A log transform does the trick.


    # First, remove problematic stocks: 3 cases for 2010; 0 cases for 2011
    remove <- which(apply(vols, 1, min) < 0.004)
    if (length(remove) > 0) vols <- vols[-remove, ]
    vols <- log(vols)
    means <- apply(vols, 2, mean)
    sds <- apply(vols, 2, sd)
    plot(means, sds, xlab = "Level", ylab = "Spread", main = "Log Transformed Data")


{:.center}
![plot of chunk unnamed-chunk-3](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-3.png) 



### Smoothing the Profiles

Clearly, we expect neighboring data points of a volatility profile to be very close to each other. In such cases as this, one can often improve the quality of one's data by letting neighboring points "inform" each other. With a little thought and exploration, I found that a parametric fit worked nicely to smooth the volatility profiles. The 78 data points are well summarized by a fifth degree polynomial.


    # Initialize
    stocks <- rownames(vols)
    n <- nrow(vols)
    m <- ncol(vols)
    t.mat <- cbind(t, t^2, t^3, t^4, t^5)
    
    # Demonstrate fit on random stock
    set.seed(2)
    i <- runif(1, 1, n)
    v <- unlist(vols[i, ])
    L <- lm(v ~ t.mat)
    
    # Plot original points with fittted curve
    plot(t, v, main = stocks[i], xlab = "Minutes into Day",
         ylab = "Log of Squared Volatility")
    lines(t, L$fit, col = 2)


{:.center}
![plot of chunk unnamed-chunk-4](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-4.png) 


The residuals show no remaining structure in the data.


    plot(t, L$res, main = stocks[i], xlab = "Minutes into Day",
         ylab = "Log of Squared Volatlity (Residual)")
    abline(h = 0, col = 2, lty = 2)


{:.center}
![plot of chunk unnamed-chunk-5](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-5.png) 


I apply this smoothing procedure to each stock.


    smooth <- matrix(NA, n, m, dimnames = list(stocks, t))
    for (i in 1:n) {
        v <- unlist(vols[i, ])
        L <- lm(v ~ t.mat)
        smooth[i, ] <- L$fit
    }
    
    res <- vols - smooth
    dimnames(res) <- list(stocks, t)
    plot(t, apply(res, 2, mean), main = "Average of Residuals", xlab = "Mean",
         ylab = "Minutes into Day")
    abline(h = 0, col = 2, lty = 2)


{:.center}
![plot of chunk unnamed-chunk-6](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-6.png) 


Notice that the averages of the residuals do not show any discernable pattern either.


### Principal Component Analysis

It is still not obvious how we should compare the volatility profiles to each other. Each profile consists of 78 highly correlated values, and they all have a very similar shape to their curves. In other words, 78 values seems like it's probably overkill. Is there some simpler representation of each profile that still captures the variation among them? To me, this seems like a perfect chance to make use of [principal component analysis](https://en.wikipedia.org/wiki/Principal_component_analysis).


    p <- princomp(smooth)
    vars <- p$sdev^2
    varprops <- vars/sum(vars)
    signif(varprops[1:8], 3)


    ##   Comp.1   Comp.2   Comp.3   Comp.4   Comp.5   Comp.6   Comp.7   Comp.8 
    ## 9.58e-01 1.61e-02 8.52e-03 7.09e-03 5.59e-03 4.63e-03 1.17e-16 1.12e-16


    plot(1:8, varprops[1:8], type = "s", ylab = "Proportion of Variability Gained",
         xlab = "Number of Principal Components", main = "Principal Components")
    abline(h = 0, lty = 2, col = 3)


{:.center}
![plot of chunk unnamed-chunk-7](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-7.png) 


Indeed, nearly ninety six percent of the variability is concentrated in the first principal component. I will maintain the second principal component as well, largely because it allows for richer plots. These two principal components are a proxy for the volatility profiles. Below is a plot of the "landscape" of these two components. For my analysis, I'd like to focus on typical stocks, so I decided to only keep the stocks that are within two and a half standard deviations of the median point, which is the *vast* majority of them.


    plot(p$scores[, 1], p$scores[, 2], pch = ".", xlab = "First Principal Component", 
         ylab = "Second Principal Component")
    m1 <- median(p$scores[, 1])
    m2 <- median(p$scores[, 2])
    points(m1, m2, col = 2, pch = 20)
    sd1 <- sd(p$scores[, 1])
    sd2 <- sd(p$scores[, 2])
    abline(v = m1 - 2.5 * sd1, col = 4, lty = 2)
    abline(v = m1 + 2.5 * sd1, col = 4, lty = 2)
    abline(h = m2 - 2.5 * sd2, col = 4, lty = 2)
    abline(h = m2 + 2.5 * sd2, col = 4, lty = 2)


{:.center}
![plot of chunk unnamed-chunk-8](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-8.png) 

    
    inside <- which(p$scores[, 1] > m1 - 2.5 * sd1 & p$scores[, 1] < m1 + 2.5 * sd1
                    & p$scores[, 2] > m2 - 2.5 * sd2 & p$scores[, 2] < m2 + 2.5 * sd2)
    smooth <- smooth[inside, ]



At this point, we should also following the same steps on the 2011 data. This way, we can finalize a common subset of stocks to keep throughout the rest of the analysis.


    vols <- read.csv("2011.csv", row.names = 1)
    t <- 2.5 + 5 * (0:77)
    ## Log transform
    remove <- which(apply(vols, 1, min) < 0.004)
    if (length(remove) > 0) vols <- vols[-remove, ]
    vols <- log(vols)
    ## Initialize
    stocks <- rownames(vols)
    n <- nrow(vols)
    m <- ncol(vols)
    t.mat <- cbind(t, t^2, t^3, t^4, t^5)
    ## Smooth
    smooth2011 <- matrix(NA, n, m, dimnames = list(stocks, t))
    for (i in 1:n) {
        v <- unlist(vols[i, ])
        L <- lm(v ~ t.mat)
        smooth2011[i, ] <- L$fit
    }
    ## Principal component analysis
    p2011 <- princomp(smooth2011)
    vars <- p2011$sdev^2
    varprops <- vars/sum(vars)
    signif(varprops[1:8], 3)


    ##   Comp.1   Comp.2   Comp.3   Comp.4   Comp.5   Comp.6   Comp.7   Comp.8 
    ## 9.69e-01 1.38e-02 6.77e-03 3.91e-03 3.69e-03 2.47e-03 3.93e-16 1.48e-16


    m1 <- median(p2011$scores[, 1])
    m2 <- median(p2011$scores[, 2])
    sd1 <- sd(p2011$scores[, 1])
    sd2 <- sd(p2011$scores[, 2])
    
    # Remove atypical stocks
    inside <- which(p2011$scores[, 1] > m1 - 2.5 * sd1 & p2011$scores[, 1] < m1 + 2.5 * sd1
                    & p2011$scores[, 2] > m2 - 2.5 * sd2 & p2011$scores[, 2] < m2 + 2.5 * sd2)
    smooth2011 <- smooth2011[inside, ]
    
    # Keep stocks that are common to both years
    stocks <- intersect(rownames(smooth), rownames(smooth2011))
    smooth <- smooth[rownames(smooth) %in% stocks, ]
    smooth2011 <- smooth2011[rownames(smooth2011) %in% stocks, ]



Excluding these stocks may have some impact on the principal components, so I need to run that analysis once more for each year.


    p <- princomp(smooth)
    vars <- p$sdev^2
    varprops <- vars/sum(vars)
    signif(varprops[1:8], 3)


    ##   Comp.1   Comp.2   Comp.3   Comp.4   Comp.5   Comp.6   Comp.7   Comp.8 
    ## 9.58e-01 1.46e-02 8.93e-03 7.67e-03 5.93e-03 4.69e-03 3.84e-16 5.21e-17


    
    p2011 <- princomp(smooth2011)
    vars <- p2011$sdev^2
    varprops <- vars/sum(vars)
    signif(varprops[1:8], 3)


    ##   Comp.1   Comp.2   Comp.3   Comp.4   Comp.5   Comp.6   Comp.7   Comp.8 
    ## 9.68e-01 1.46e-02 6.80e-03 4.05e-03 3.80e-03 2.44e-03 1.91e-16 5.64e-17



Let us try to interpret these principal components by visualizing the first and second loadings for 2010. It seems that the first principal component is basically measuring the overall level of the volatility curves, except weighting the the end of the day less than the rest.


    plot(colnames(smooth), p$loading[, 1], xlab = "Minutes into Day",
         ylab = "Weight in First Principal Component", main = "2010")


{:.center}
![plot of chunk unnamed-chunk-11](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-11.png) 


The second principal component acts as a measure of the difference between late and early volatility.


    plot(colnames(smooth), p$loading[, 2], xlab = "Minutes into Day",
         ylab = "Weight in Second Principal Component", main = "2010")


{:.center}
![plot of chunk unnamed-chunk-12](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-12.png) 


Before we proceed, let's compare this to the 2011 data. As long as there weren't any major systematic changes in the stock market during the interim, we might expect the 2011 dataset to have similar eigenvectors to the 2010 dataset. If they are similar enough, then the principal components are measuring basically the same thing. That would be nice, because it would make comparing volatilites across the two periods much more straightforward.

The first and second loadings of the 2011 principal component analysis are plotted below. They definitely bear some resemblance to their 2010 counterparts.


    plot(colnames(smooth2011), p2011$loading[, 1], xlab = "Minutes into Day",
         ylab = "Weight in First Principal Component", main = "2011")


{:.center}
![plot of chunk unnamed-chunk-13](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-13.png) 

    
    # Angle between loadings
    acos(sum(p$loading[, 1] * p2011$loading[, 1])) # lengths are both 1


    ## [1] 0.05283



In fact the angle between this loading and the 2010 loading is only about .05 radians. If we take use the average of the two loadings, each is only being shifted by half that angle. This is a minor enough shift that it should have a negligible effect on our variability.


    plot(colnames(smooth2011), p2011$loading[, 2], xlab = "Minutes into Day",
         ylab = "Weight in Second Principal Component", main = "2011")


{:.center}
![plot of chunk unnamed-chunk-14](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-14.png) 


The second principal components both consist of the late volatility minus the early volatility. However, the 2010 version weights the very early volatility heavily, while the 2011 version gives more weight to the very late volatility. They are not nearly as similar to each other as the first principal component loadings were, but I will still take their average. Remember, the second principal components do not contain all that much variability anyway; they are primarily being kept for plotting purposes. Note that this average is approximately perpendicular to the averaged first principal component loading.

This may seem like a hassle, but I think it is worth it to maintain consistency between both periods. Using the exact same principal component coordinate system for each year will make comparisons much simpler in the end. It will allow us to plot all volatility profiles in the same picture.


    loading1 <- (p2011$loading[, 1] + p$loading[, 1])/2
    loading1 <- loading1/sum(loading1^2)
    loading2 <- (p2011$loading[, 2] + p$loading[, 2])/2
    loading2 <- loading2/sum(loading2^2)
    
    # Compute new principal component values
    pc1.2010 <- t(loading1 %*% (t(smooth) - p$center)) + sum(p$center * loading1)
    pc2.2010 <- t(loading2 %*% (t(smooth) - p$center)) + sum(p$center * loading2)
    pc1.2011 <- t(loading1 %*% (t(smooth2011) - p2011$center))
                + sum(p2011$center * loading1)
    pc2.2011 <- t(loading2 %*% (t(smooth2011) - p2011$center))
                + sum(p2011$center * loading2)



The volatility profiles in 2011 are in the same general vicinity as those in 2010, although they are a little different on average. In the plot below, the red points correspond to 2010 volatility profiles, and the green points correspond to 2011.


    plot(pc1.2010, pc2.2010, col = 2, pch = ".", xlab = "First Principal Component", 
         ylab = "Second Principal Component")
    points(pc1.2011, pc2.2011, col = 3, pch = ".")


{:.center}
![plot of chunk unnamed-chunk-16](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-16.png) 



### Controlling for Other Factors

Ultimately, we want to know whether differences in volatility profiles are related to whether or not a stock is in the SSCB group. But the SSCB group is systematically different from the other in a few ways. Two differences that I think are important are market cap (MC) and trading frequency (TF). Before comparing the groups, I will try to control for these factors.

#### Market Cap

First, let us see if market cap has any effect on volatility profiles by plotting our two principal components against it. There is a clear positive relationship between market cap and the first principal component.


    cap <- read.csv("cap.csv", row.names = 1, check.names = F)
    cap <- cap[rownames(cap) %in% stocks, ]
    cap.log <- log(cap[, 1])
    plot(cap.log, pc1.2010, main = "2010", xlab = "Log of Market Cap",
         ylab = "First Principal Component")
    # Because we found a relationship, we should control for it
    x.mat <- cbind(cap.log, cap.log^2)
    L <- lm(pc1.2010 ~ x.mat)
    grid <- seq(19, 26, by = 0.1)
    lines(grid, cbind(1, grid, grid^2) %*% L$coef, col = 2)


{:.center}
![plot of chunk unnamed-chunk-17](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-17.png) 

    summary(L)


    ## 
    ## Call:
    ## lm(formula = pc1.2010 ~ x.mat)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -12.732  -3.408  -0.153   3.198  13.928 
    ## 
    ## Coefficients:
    ##               Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)  -233.2789    21.4037  -10.90   <2e-16 ***
    ## x.matcap.log   20.4958     1.9168   10.69   <2e-16 ***
    ## x.mat          -0.4099     0.0428   -9.58   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
    ## 
    ## Residual standard error: 4.8 on 1622 degrees of freedom
    ## Multiple R-squared: 0.319,	Adjusted R-squared: 0.318 
    ## F-statistic:  379 on 2 and 1622 DF,  p-value: <2e-16



Plotting the residuals, we see that no clear pattern remains. Therefore, I am satisfied that we have controlled for market cap.


    plot(cap.log, L$res, main = "2010", xlab = "Log of Market Cap",
         ylab = "Residuals of First Principal Component")


{:.center}
![plot of chunk unnamed-chunk-18](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-18.png) 

    pc1.2010 <- L$res + mean(pc1.2010)



Likewise for 2011.


    plot(cap.log, pc1.2011, main = "2011", xlab = "Log of Market Cap",
         ylab = "First Principal Component")
    L <- lm(pc1.2011 ~ x.mat)
    lines(grid, cbind(1, grid, grid^2) %*% L$coef, col = 2)


{:.center}
![plot of chunk unnamed-chunk-19](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-19.png) 

    summary(L)


    ## 
    ## Call:
    ## lm(formula = pc1.2011 ~ x.mat)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -14.983  -2.895  -0.157   2.762  13.372 
    ## 
    ## Coefficients:
    ##               Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)  -229.2398    19.4672   -11.8   <2e-16 ***
    ## x.matcap.log   19.6355     1.7434    11.3   <2e-16 ***
    ## x.mat          -0.3853     0.0389    -9.9   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
    ## 
    ## Residual standard error: 4.36 on 1622 degrees of freedom
    ## Multiple R-squared: 0.402,	Adjusted R-squared: 0.401 
    ## F-statistic:  546 on 2 and 1622 DF,  p-value: <2e-16




    plot(cap.log, L$res, main = "2011", xlab = "Log of Market Cap",
         ylab = "Residuals of First Principal Component")


{:.center}
![plot of chunk unnamed-chunk-20](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-20.png) 

    pc1.2011 <- L$res + mean(pc1.2011)




The second principal component is weakly related to market cap.


    plot(cap.log, pc2.2010, main = "2010", xlab = "Log of Market Cap",
         ylab = "Second Principal Component")
    L <- lm(pc2.2010 ~ x.mat)
    lines(grid, cbind(1, grid, grid^2) %*% L$coef, col = 2)


{:.center}
![plot of chunk unnamed-chunk-21](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-21.png) 

    summary(L)


    ## 
    ## Call:
    ## lm(formula = pc2.2010 ~ x.mat)
    ## 
    ## Residuals:
    ##    Min     1Q Median     3Q    Max 
    ## -2.428 -0.503 -0.005  0.495  2.713 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)   
    ## (Intercept)   4.29506    3.32191    1.29   0.1962   
    ## x.matcap.log -0.82055    0.29749   -2.76   0.0059 **
    ## x.mat         0.02106    0.00664    3.17   0.0015 **
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
    ## 
    ## Residual standard error: 0.745 on 1622 degrees of freedom
    ## Multiple R-squared: 0.0571,	Adjusted R-squared: 0.056 
    ## F-statistic: 49.1 on 2 and 1622 DF,  p-value: <2e-16




    plot(cap.log, L$res, main = "2010", xlab = "Log of Market Cap",
         ylab = "Residuals of Second Principal Component")


{:.center}
![plot of chunk unnamed-chunk-22](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-22.png) 

    pc2.2010 <- L$res + mean(pc2.2010)




    plot(cap.log, pc2.2011, main = "2011", xlab = "Log of Market Cap",
         ylab = "Second Principal Component")
    L <- lm(pc2.2011 ~ x.mat)
    lines(grid, cbind(1, grid, grid^2) %*% L$coef, col = 2)


{:.center}
![plot of chunk unnamed-chunk-23](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-23.png) 

    summary(L)


    ## 
    ## Call:
    ## lm(formula = pc2.2011 ~ x.mat)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -2.0889 -0.5094 -0.0186  0.4772  1.9213 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)  17.32703    3.11140    5.57  3.0e-08 ***
    ## x.matcap.log -1.70005    0.27864   -6.10  1.3e-09 ***
    ## x.mat         0.03523    0.00622    5.66  1.8e-08 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
    ## 
    ## Residual standard error: 0.698 on 1622 degrees of freedom
    ## Multiple R-squared: 0.0772,	Adjusted R-squared: 0.076 
    ## F-statistic: 67.8 on 2 and 1622 DF,  p-value: <2e-16




    plot(cap.log, L$res, main = "2011", xlab = "Log of Market Cap",
         ylab = "Residuals of Second Principal Component")


{:.center}
![plot of chunk unnamed-chunk-24](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-24.png) 

    pc2.2011 <- L$res + mean(pc2.2011)



Finally, just to be safe, let's glance at the two-dimensional picture to make sure there are no strange interaction effects. In the plots below, the higher market cap stocks are green, while the lower cap stocks are red. I see no patterns here.


    high <- cap.log > median(cap.log)
    cols <- as.integer(high) + 2
    plot(pc1.2010, pc2.2010, pch = ".", col = cols, cex = 1.2, main = "2010",
         xlab = "First Principal Component (MC Removed)", 
         ylab = "Second Principal Component (MC Removed)")


{:.center}
![plot of chunk unnamed-chunk-25](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-25.png) 



    plot(pc1.2011, pc2.2011, pch = ".", col = cols, cex = 1.2, main = "2011",
         xlab = "First Principal Component (MC Removed)", 
         ylab = "Second Principal Component (MC Removed)")


{:.center}
![plot of chunk unnamed-chunk-26](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-26.png) 


#### Trading Frequency

Next, we will consider the effect of trading frequency on the principal components. We must first remove the relationship between trading frequency and market cap.


    act <- read.csv("activity.csv", row.names = 1)
    act <- act[rownames(act) %in% stocks, ]
    act.logmean <- log(apply(act, 1, mean))
    plot(cap.log, act.logmean, xlab = "Log of Market Cap",
         ylab = "Log of Avg of Number of Seconds in which Trades Occurred")
    L <- lm(act.logmean ~ x.mat)
    lines(grid, cbind(1, grid, grid^2) %*% L$coef, col = 2)


{:.center}
![plot of chunk unnamed-chunk-27](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-27.png) 

    summary(L)


    ## 
    ## Call:
    ## lm(formula = act.logmean ~ x.mat)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -2.1483 -0.3083  0.0492  0.3412  1.3259 
    ## 
    ## Coefficients:
    ##               Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)  -11.84421    2.13106   -5.56  3.2e-08 ***
    ## x.matcap.log   1.47716    0.19085    7.74  1.7e-14 ***
    ## x.mat         -0.02605    0.00426   -6.11  1.2e-09 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
    ## 
    ## Residual standard error: 0.478 on 1622 degrees of freedom
    ## Multiple R-squared: 0.47,	Adjusted R-squared: 0.47 
    ## F-statistic:  720 on 2 and 1622 DF,  p-value: <2e-16



The fit seems fine, so I will use it to control.


    plot(cap.log, L$res, main = "2010", xlab = "Log of Market Cap",
         ylab = "Residuals of Log of Avg Number of Seconds in which Trades Occurred")


{:.center}
![plot of chunk unnamed-chunk-28](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-28.png) 

    act.res <- L$res + mean(act.logmean)



Now, I can look for a relationship between the principal components and the trading frequency. There is a major effect on the first principal component.


    plot(act.res, pc1.2010, main = "2010",
         ylab = "First Principal Component (MC Removed)", 
         xlab = "Log of Avg Number of Seconds in which Trades Occurred (MC Removed)")
    x.mat <- cbind(act.res, act.res^2)
    L <- lm(pc1.2010 ~ x.mat)
    grid <- seq(6.5, 9, by = 0.1)
    lines(grid, cbind(1, grid, grid^2) %*% L$coef, col = 2)


{:.center}
![plot of chunk unnamed-chunk-29](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-29.png) 

    summary(L)


    ## 
    ## Call:
    ## lm(formula = pc1.2010 ~ x.mat)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -13.577  -3.063  -0.238   2.766  15.928 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)  -122.857     19.455   -6.31  3.5e-10 ***
    ## x.matact.res   40.345      4.963    8.13  8.4e-16 ***
    ## x.mat          -2.830      0.316   -8.96  < 2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
    ## 
    ## Residual standard error: 4.28 on 1622 degrees of freedom
    ## Multiple R-squared: 0.204,	Adjusted R-squared: 0.203 
    ## F-statistic:  208 on 2 and 1622 DF,  p-value: <2e-16



A quadratic fit controls for this.


    plot(act.res, L$res, main = "2010",
         ylab = "Residuals of First Principal Component (MC Removed)", 
         xlab = "Log of Avg Number of Seconds in which Trades Occurred (MC Removed)")


{:.center}
![plot of chunk unnamed-chunk-30](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-30.png) 

    pc1.2010 <- L$res + mean(pc1.2010)




    plot(act.res, pc1.2011, main = "2011",
         ylab = "First Principal Component (MC Removed)", 
         xlab = "Log of Avg Number of Seconds in which Trades Occurred (MC Removed)")
    L <- lm(pc1.2011 ~ x.mat)
    lines(grid, cbind(1, grid, grid^2) %*% L$coef, col = 2)


{:.center}
![plot of chunk unnamed-chunk-31](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-31.png) 

    summary(L)


    ## 
    ## Call:
    ## lm(formula = pc1.2011 ~ x.mat)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -13.235  -2.498  -0.278   2.222  12.710 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)   -87.883     17.525   -5.01  5.9e-07 ***
    ## x.matact.res   30.541      4.470    6.83  1.2e-11 ***
    ## x.mat          -2.200      0.285   -7.73  1.9e-14 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
    ## 
    ## Residual standard error: 3.86 on 1622 degrees of freedom
    ## Multiple R-squared: 0.219,	Adjusted R-squared: 0.218 
    ## F-statistic:  228 on 2 and 1622 DF,  p-value: <2e-16




    plot(act.res, L$res, main = "2011",
         ylab = "Residuals of First Principal Component (MC Removed)", 
         xlab = "Log of Avg Number of Seconds in which Trades Occurred (MC Removed)")


{:.center}
![plot of chunk unnamed-chunk-32](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-32.png) 

    pc1.2011 <- L$res + mean(pc1.2011)



There is basically no relationship between trading frequency and the second principal component in 2010. However, there is a weak relationship in the 2011 dataset. To maintain consistency between the two data sets, I used a quadratic fit below. It has almost no effect on the 2010 data.


    plot(act.res, pc2.2010, main = "2010",
         ylab = "Second Principal Component (MC Removed)", 
         xlab = "Log of Avg Number of Seconds in which Trades Occurred (MC Removed)")
    L <- lm(pc2.2010 ~ x.mat)
    lines(grid, cbind(1, grid, grid^2) %*% L$coef, col = 2)


{:.center}
![plot of chunk unnamed-chunk-33](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-33.png) 

    summary(L)


    ## 
    ## Call:
    ## lm(formula = pc2.2010 ~ x.mat)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -2.4545 -0.5097 -0.0164  0.4913  2.6827 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)   
    ## (Intercept)   -8.8887     3.3800   -2.63   0.0086 **
    ## x.matact.res   1.4151     0.8622    1.64   0.1009   
    ## x.mat         -0.0928     0.0549   -1.69   0.0910 . 
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
    ## 
    ## Residual standard error: 0.744 on 1622 degrees of freedom
    ## Multiple R-squared: 0.00246,	Adjusted R-squared: 0.00123 
    ## F-statistic:    2 on 2 and 1622 DF,  p-value: 0.135




    plot(act.res, L$res, main = "2010",
         ylab = "Residuals of Second Principal Component (MC Removed)", 
         xlab = "Log of Avg Number of Seconds in which Trades Occurred (MC Removed)")


{:.center}
![plot of chunk unnamed-chunk-34](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-34.png) 

    pc2.2010 <- L$res + mean(pc2.2010)




    plot(act.res, pc2.2011, main = "2011",
         ylab = "Second Principal Component (MC Removed)", 
         xlab = "Log of Avg Number of Seconds in which Trades Occurred (MC Removed)")
    L <- lm(pc2.2011 ~ x.mat)
    lines(grid, cbind(1, grid, grid^2) %*% L$coef, col = 2)


{:.center}
![plot of chunk unnamed-chunk-35](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-35.png) 

    summary(L)


    ## 
    ## Call:
    ## lm(formula = pc2.2011 ~ x.mat)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -2.0023 -0.4839 -0.0414  0.4426  2.4732 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)  -13.5292     3.1052   -4.36  1.4e-05 ***
    ## x.matact.res   2.9747     0.7921    3.76  0.00018 ***
    ## x.mat         -0.2058     0.0504   -4.08  4.7e-05 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
    ## 
    ## Residual standard error: 0.683 on 1622 degrees of freedom
    ## Multiple R-squared: 0.0403,	Adjusted R-squared: 0.0391 
    ## F-statistic: 34.1 on 2 and 1622 DF,  p-value: 3.22e-15




    plot(act.res, L$res, main = "2011",
         ylab = "Residuals of Second Principal Component (MC Removed)", 
         xlab = "Log of Avg Number of Seconds in which Trades Occurred (MC Removed)")


{:.center}
![plot of chunk unnamed-chunk-36](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-36.png) 

    pc2.2011 <- L$res + mean(pc2.2011)



There are no apparent interactions.


    high <- act.res > median(act.res)
    cols <- as.integer(high) + 2
    plot(pc1.2010, pc2.2010, pch = ".", col = cols, cex = 1.2, main = "2010",
         xlab = "First Principal Component (MC and TF Removed)", 
         ylab = "Second Principal Component (MC and TF Removed)")


{:.center}
![plot of chunk unnamed-chunk-37](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-37.png) 



    plot(pc1.2011, pc2.2011, pch = ".", col = cols, cex = 1.2, main = "2011",
         xlab = "First Principal Component (MC and TF Removed)", 
         ylab = "Second Principal Component (MC and TF Removed)")


{:.center}
![plot of chunk unnamed-chunk-38](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-38.png) 



### Comparing SSCB Stocks to non-SSCB Stocks

It's worth pointing out that all of the decisions made above were determined before I saw any results by group (SSCB vs non-SSCB). I say this because the decisions leading up to this involved a lot of "wiggle room." An inept (or dishonest) data analyst can often exploit such wiggle room to find "positive" results where none exist. This is an honest analysis in search of true answers, even if they are null.

It is clear from the landscape that there are obvious differences between the SSCB (green) and non-SSCB (red) stocks in both 2010 and 2011.


    SSCBstocks <- readLines("RussellOrSP.txt")
    sscb <- stocks %in% SSCBstocks
    cols <- as.integer(sscb) + 2
    plot(pc1.2010, pc2.2010, pch = ".", col = cols, cex = 1.2, main = "2010",
         xlab = "First Principal Component (MC and TF Removed)", 
         ylab = "Second Principal Component (MC and TF Removed)")
    
    m1n0 <- mean(pc1.2010[!sscb])
    m2n0 <- mean(pc2.2010[!sscb])
    m1s0 <- mean(pc1.2010[sscb])
    m2s0 <- mean(pc2.2010[sscb])    
    points(m1n0, m2n0, col = 2, pch = "N")
    points(m1s0, m2s0, col = 3, pch = "S")


{:.center}
![plot of chunk unnamed-chunk-39](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-39.png) 



    plot(pc1.2011, pc2.2011, pch = ".", col = cols, cex = 1.2, main = "2011",
         xlab = "First Principal Component (MC and TF Removed)", 
         ylab = "Second Principal Component (MC and TF Removed)")
    
    m1n1 <- mean(pc1.2011[!sscb])
    m2n1 <- mean(pc2.2011[!sscb])
    m1s1 <- mean(pc1.2011[sscb])
    m2s1 <- mean(pc2.2011[sscb])
    points(m1n1, m2n1, col = 2, pch = "N")
    points(m1s1, m2s1, col = 3, pch = "S")


{:.center}
![plot of chunk unnamed-chunk-40](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-40.png) 


Finally, the plot below shows locations and concentration ellipses of both groups of stocks in both periods.


    s1n0 <- sd(pc1.2010[!sscb])
    s2n0 <- sd(pc2.2010[!sscb])
    s1s0 <- sd(pc1.2010[sscb])
    s2s0 <- sd(pc2.2010[sscb])
    s1n1 <- sd(pc1.2011[!sscb])
    s2n1 <- sd(pc2.2011[!sscb])
    s1s1 <- sd(pc1.2011[sscb])
    s2s1 <- sd(pc2.2011[sscb])
    
    library(plotrix)
    
    plot(c(10, 25), c(-4.5, -1.5), type = "n", col = cols, cex = 1.2,
         main = "Both Periods", 
         xlab = "First Principal Component (MC and TF Removed)",
         ylab = "Second Principal Component (MC and TF Removed)")
    
    draw.ellipse(m1n0, m2n0, 0.5 * s1n0, 0.5 * s2n0, border = 2, lty = 3)
    draw.ellipse(m1s0, m2s0, 0.5 * s1s0, 0.5 * s2s0, border = 3, lty = 3)
    draw.ellipse(m1n1, m2n1, 0.5 * s1n1, 0.5 * s2n1, border = 2, lty = 3)
    draw.ellipse(m1s1, m2s1, 0.5 * s1s1, 0.5 * s2s1, border = 3, lty = 3)
    
    text(m1n0, m2n0, "N0", col = 2)
    text(m1s0, m2s0, "S0", col = 3)
    text(m1n1, m2n1, "N1", col = 2)
    text(m1s1, m2s1, "S1", col = 3)


{:.center}
![plot of chunk unnamed-chunk-41](/static/2013-01-14-single-stock-circuit-breakers/unnamed-chunk-41.png) 


The picture shows no evidence that the SSCB rules have changed the affected stocks' volatility profiles in any important way. Of course, lingering systematic differences between the groups could be distorting our results. If the group of stocks to get the SSCB rules had been selected randomly, then perhaps we could we could be more confident in our conclusions. Unfortunately, that's not how the rules were implemented, and so I did my best considering the circumstances.

One other notable observation is that volatility profiles in general are shifting over time. That might be an interesting topic to explore further at some point.



