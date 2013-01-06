---
layout: post
title: Including Python Code in an R Package
category: tutorial
tags: [r, python]
---
{% include JB/setup %}

Often it is useful to use other programming languages within an R package. In particular, some things can be done more easily with Python than with R. Personally, I prefer Python for web scraping and for dealing with files. When using Python with R, I don't recommend trying to pass data back and forth directly between the two; I don't know of any simple way to do this. Instead, let the two languages communicate via external files that they create or modify. This paper, along with the accompanying package Pydemo, are intended to provide guidance for beginners who wish to use Python code with R. The Pydemo package includes minimal working Python code that interacts successfully with R. The [Pydemo.tar](/static/Pydemo.tar) archive contains the original set of files and folders that are built up into the full Pydemo package.

The process described below was completed on the Ubuntu 11.04 operating system. Likely, the same process will work for other Linux releases, as well as for Mac OS. However, the Windows operating system is quite different, and these instruction probably won't work in that case.

Also, note that these instructions should work for pretty much any scripting language, not just Python. However, keep in mind that the user may have a different version of the language than you are expecting, so that can cause your package to break.


## A Look at the Pydemo Package

The **inst** folder within the package's main directory is where any Python code belongs. In Pydemo, the Python code is in the file **Pygreet.py**. It takes one argument and creates a file with a greeting message.

    import sys
    filename = "Pygreetings%s.txt" % sys.argv[1]
    ofile = open(filename, "a")
    ofile.write("Hello %s, this message is brought to you by Python!\n" % sys.argv[1])
    ofile.close()
    print "File created: %s" % filename

Now R code within this package can run the python script **Pygreet.py**, as shown below. Within the **R** folder, we have a file **greet.R** with the code

    greet <- function(name) {
      path <- paste(system.file(package="Pydemo"), "Pygreet.py", sep="/")
      command <- paste("python", path, name)
      system(command)
    }

Finally, the package has a **NAMESPACE** file and a **DESCRIPTION** file, and the folder **man** contains two simple documentation files. All of these files can be found in [Pydemo.tar](/static/Pydemo.tar). To finish building your package, navigate a terminal session to the parent of your package's main directory and run

`> R CMD check Pydemo`

`> R CMD build Pydemo`

This creates **Pydemo\_1.0.tar.gz** which is ready to be installed and used on any computer that has Python. Good luck!

