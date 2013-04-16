---
layout: post
title: Spline Derivatives
category: tutorial
tags: [r]
---
{% include JB/setup %}





I recently assisted a physicist with a simple data analysis issue. She had taken a series of measurements with an instrument at varying levels of some setting $\Omega$. The consecutive data points should be noisy about an underlying smooth curve. The physicist wanted to estimate the derivative of this curve at her data points.

Even without assuming any simple parametric structure to the curve, we can achieve this quite easily by using R's built-in `smooth.spline` function.

The [data](/static/NaCl_Omega.ReZ.ImZ.csv) has three columns: the predictor variable $\Omega$, and two columns (corresponding to real and imaginary parts) of response variables. The different values of $\Omega$ are spaced out evenly on a logarithmic scale. By log-transforming these $\Omega$ values, we can get evenly spaced data points; this will make the spline fit work better.


{% highlight r %}
filename <- "NaCl_Omega.ReZ.ImZ.csv"
X <- read.csv(filename, header = F)
O <- X[, 1]
R <- X[, 2]
I <- X[, 3]

# Transform Omega values
o <- log(O)

### Reals ###

# Fit smooth curve for Reals
par(mfrow = c(1, 2))
plot(o, R, xlab = expression(log[n] ~ Omega), ylab = "Real",
     main = "Real Data with Spline Fit")
s <- smooth.spline(o, R)
lines(predict(s, o), col = 2, lwd = 2)

plot(O, R, xlab = expression(Omega), ylab = "Real",
     main = "Data with Spline Fit")
lines(O, s$y, col = 2, lwd = 2)
{% endhighlight %}

{:.center}
![plot of chunk unnamed-chunk-2](/static/2013-04-03-spline-derivatives/unnamed-chunk-2.png) 


Now, the `predict.smooth.spline` method can give us the derivative of the fit curve at our chosen values. However, because of our natural logarithm transformation, these will be the derivatives with respect to $\log \Omega$. To get the derivatives with respect to $\Omega$, we simply need to divide these derivatives by $\Omega$ because


<div>\begin{align*}
\frac{d R}{d \Omega} &= \left( \frac{d R}{d \log \Omega} \right) \left( \frac{d \log \Omega}{d \Omega} \right)\\
 &= \left( \frac{d R}{d \log \Omega} \right) \left( \frac{1}{\Omega} \right)
\end{align*}</div>


{% highlight r %}
# Numerical derivatives
dy <- predict(s, o, deriv = 1)$y/O
outR <- log10(-dy)
plot(log10(O/2/pi), outR, xlab = expression(log[10] ~ Frequency),
     ylab = expression(log[10] ~ "(-d Real)"),
     main = "Derivative of Real Spline Fit")
{% endhighlight %}

{:.center}
![plot of chunk unnamed-chunk-3](/static/2013-04-03-spline-derivatives/unnamed-chunk-3.png) 


Repeat for imaginary values.


{% highlight r %}
### Imaginaries ###

# Fit smooth curve for Imaginaries
par(mfrow = c(1, 2))
plot(o, I, xlab = expression(log[n] ~ Omega), ylab = "Imaginary",
     main = "Imaginary Data with Spline Fit")
s <- smooth.spline(o, I)
lines(o, s$y, col = 2, lwd = 2)

plot(O, I, xlab = expression(Omega), ylab = "Imaginary",
     main = "Data with Spline Fit")
lines(O, s$y, col = 2, lwd = 2)
{% endhighlight %}

{:.center}
![plot of chunk unnamed-chunk-4](/static/2013-04-03-spline-derivatives/unnamed-chunk-4.png) 



{% highlight r %}
# Numerical derivatives
dy <- predict(s, o, deriv = 1)$y/O
outI <- log10(dy)
plot(log10(O/2/pi), outI, xlab = expression(log[10] ~ Frequency),
     ylab = expression(log[10] ~ "(d Imaginary)"),
     main = "Derivative of Imaginary Spline Fit")
{% endhighlight %}

{:.center}
![plot of chunk unnamed-chunk-5](/static/2013-04-03-spline-derivatives/unnamed-chunk-5.png) 



### Automating it with Rscript

The physicist will be gathering much more data of this type. The following R script (named **ORIspline.R**) will allow her to compute the derivative estimates easily on that future data.

Example command line usage: `Rscript ORIspline.R NaCl_Omega.ReZ.ImZ.csv`

Notice that the name of the file to be analyzed is passed in as a command-line parameter.


{% highlight r %}
filename <- commandArgs(T)[1]

X <- read.csv(filename, header = F)
O <- X[, 1]
R <- X[, 2]
I <- X[, 3]

# Transform Omega values
o <- log(O)

### Reals ###
s <- smooth.spline(o, R)
dy <- predict(s, o, deriv = 1)$y/O
outR <- log10(-dy)

### Imaginaries ###
s <- smooth.spline(o, I)
dy <- predict(s, o, deriv = 1)$y/O
outI <- log10(dy)

# Create csv file
final <- cbind(O, outR, outI)
colnames(final) <- c("Omega", "log10 -dRe", "log10 dIm")
write.csv(final, paste0("derivatives_", filename), row.names = F)
{% endhighlight %}


