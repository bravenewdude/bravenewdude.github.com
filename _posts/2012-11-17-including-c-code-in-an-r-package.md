---
layout: post
title: Including C Code in an R Package
category: tutorial
tags: [r, c]
---
{% include JB/setup %}

Often it is useful to use other programming languages within an R package. In particular, C code can often perform computations much faster than R can, sometimes hundreds of times faster. For this reason, many people like to use C for perform certain subroutines within their R code. Fortunately, R has built-in functionality that facilitates its interface with C. This paper, along with the accompanying package Cdemo, are intended to provide guidance for beginners who wish to use C code with R. The Cdemo package includes minimal working C code that interacts successfully with R. The [Cdemo.tar](/static/Cdemo.tar) archive contains the original set of files and folders that are built up into the full Cdemo package.

The process described below was completed on the Ubuntu 11.04 operating system. Likely, the same process will work for other Linux releases, as well as for Mac OS. However, the Windows operating system is quite different, and these instruction may not work in that case.


## A Look at the Cdemo Package

The **src** folder within the package's main director is where any C code belongs. In Cdemo, the C code is in the file **Cadd.c**. It contains a function that adds 1 to each element in a vector of doubles.

    void Caddone(double *x, int *length) {
      for(int i=0; i<*length; i++) {
        x[i] = x[i]+1;
      }
    }

In our package's main directory, there is a file called simply **NAMESPACE** and containing the lines

    exportPattern(".")
    useDynLib(Cdemo)

Now R code within this package can call the C function `Caddone` by using the `.C` method, as shown below. Within the **R** folder, we have a file **addone.R** with the code

    addone <- function(x) {
      x <- as.double(x)
      .C("Caddone", DUP=F, x, length(x))
      return(x)
    }

Finally, the package has a **DESCRIPTION** file, and the folder **man** contains two simple documentation files. All of these files can be found in [Cdemo.tar](/static/Cdemo.tar). To finish building your package, navigate a terminal session to the parent of your package's main directory and run

`> R CMD check Cdemo`

`> R CMD build Cdemo`

This creates **Cdemo\_1.0.tar.gz** which is ready to be installed and used. Good luck!

