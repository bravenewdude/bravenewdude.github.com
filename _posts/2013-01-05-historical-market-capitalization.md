---
layout: post
title: Historical Market Capitalization
category: tutorial
tags: [finance, data scraping]
---
{% include JB/setup %}


According to Wikipedia, [Market Capitalization](http://en.wikipedia.org/wiki/Market_capitalization) "is the total value of the issued shares of a publicly traded company; it is equal to the share price times the number of [shares outstanding](http://en.wikipedia.org/wiki/Shares_outstanding)." For a data analysis project, I needed to determine the market cap of stocks at particular past dates. I already had the share prices for these dates, so I only needed to find the number of shares outstanding on these dates. I wrote code to scrape the web and estimate past shares outstanding.

My code estimates the number of shares outstanding on any historical date by considering the current number of shares outstanding and the splits that have occurred since that date. This procedure is definitely not perfect! Splits have a big impact on the number of shares outstanding, but [other factors](http://answers.yahoo.com/question/index?qid=20061026104715AAJUVbe) can affect the number as well. Hopefully the fluctuations caused by the other factors are relatively small. Regardless, the other factors are harder to get historical data for.

The current number of shares outstanding can be retreived from [Yahoo! Finance](http://finance.yahoo.com/). Unfortunately, this database is incomplete; in particular, many small stocks are missing. Split histories can be found at [GetSplitHistory](http://getsplithistory.com/); it was also missing many stocks.

My code uses the `XML` package and [XPath](http://en.wikipedia.org/wiki/XPath). Also, because web scraping can take a long time and is particularly vulnerable to failure, I created an `append.csv` function. It behaves much like `write.csv`, but it builds the csv file one line at a time, in order to save your progress as you go.

<a id="Outstanding"></a>

    html <- function(url) {
        # Attempts to retrieve html from url twice
        x <- NA
        try(x <- htmlParse(url), silent = T)
        if (is.na(x)) {
            Sys.sleep(5)  # Pause and try again
            try(x <- htmlParse(url), silent = T)
            if (is.na(x)) 
                print(paste("Error: Could not connect to", url))
        }
        return(x)
    }
    
    outstanding <- function(symbol, dates) {
        # Scrapes finance.yahoo.com and getsplithistory.com to determine the
        # number of outstanding shares of a stock on any given date. Works as of
        # early January 2013, but may stop working if either site changes html.
        
        # First get current number of shares from Yahoo
        yahooURL <- "http://finance.yahoo.com/q/ks?s=XXX"
        thisURL <- gsub("XXX", symbol, yahooURL)
        x <- html(thisURL)
        if (is.na(x)) 
            return(rep(NA, length(dates)))
        pattern <- "//td[preceding-sibling::td[text()='Shares Outstanding']]/text()"
        y <- xpathSApply(x, pattern, xmlValue)
        if (length(y) == 0) {
            print(paste(symbol, "Error: Yahoo has no data"))
            return(rep(NA, length(dates)))
        }
        power <- switch(substr(y, nchar(y), nchar(y)), M = 6, B = 9)
        currentshares <- as.numeric(substr(y, 1, nchar(y) - 1)) * 10^power
        
        # Next, use split history to determine past shares
        gshURL <- "http://getsplithistory.com/XXX"
        thisURL <- gsub("XXX", symbol, gshURL)
        x <- html(thisURL)
        if (is.na(x)) 
            return(rep(NA, length(dates)))
        pattern <- "//table[@id='table-splits']/tbody/tr[@class='2']"
        y <- xpathSApply(x, pattern, xmlValue)
        if (length(y) == 0) {
            pattern <- "//b[text()='404 error.']"
            y <- xpathSApply(x, pattern, xmlValue)
            if (length(y) > 0) {
                print(paste(symbol, "Error: GetSplitHistory has no data"))
                return(rep(NA, length(dates)))
            }
            # stock has never split
            return(rep(currentshares, length(dates)))
        }
        splitdates <- strptime(substr(y, 1, 12), "%b %d, %Y")
        ratios <- as.numeric(substr(y, 13, 13))/as.numeric(substr(y, 17, 17))
        results <- rep(currentshares, length(dates))
        for (i in 1:length(dates)) {
            j <- 1
            while (dates[i] <= splitdates[j]) {
                results[i] <- results[i]/ratios[j]
                if (j == length(splitdates)) 
                    break
                j <- j + 1
            }
        }
        return(results)
    }
    
    append.csv <- function(row, file = "myfile.csv", rowname = "") {
        # Adds one row to a csv file
        cat(rowname, row, file = file, sep = ",", append = file.exists(file))
        cat("\n", file = file, append = T)
    }
    
    
    library(XML)
    
    symbols <- c("AAPL", "MSFT", "GOOG")
    dates <- c("2009-03-01", "2010-05-17")
    
    append.csv(dates)  # creates file with header
    results <- matrix(NA, length(symbols), length(dates))
    colnames(results) <- dates
    rownames(results) <- symbols
    dates <- strptime(dates, "%Y-%m-%d")
    for (i in 1:length(symbols)) {
        results[i, ] <- outstanding(symbols[i], dates)
        append.csv(results[i, ], rowname = symbols[i])
        Sys.sleep(5)
    }



Finally, for any past date, multiply the price by number of shares outstanding to estimate a stock's market capitalization on that date.

