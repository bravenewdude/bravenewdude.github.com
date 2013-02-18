---
layout: post
title: The Importance of Random Sampling
category : opinion
tags : [opinion, college]
---
{% include JB/setup %}


{:.center}
![random sampling comic](/static/2013-02-17-the-importance-of-random-sampling/randomsampling.png) 

Obviously the "news" can't include literally everything that happens, so it has to be some subset. Ideally, the subset would be a [random sample](https://en.wikipedia.org/wiki/Simple_random_sample) of reality, but I wouldn't know how to begin taking a random sample of the set of all events that happened this week. Furthermore, obviously some events are more important than others; some events are more interesting than others; some events fit the news-propogaters' worldviews better than others.

Anyway, nonrandom news is what we get, and the result is that news-watchers are systematically misinformed. I wouldn't say that people are being massively "propogandized." That implies that the slant is *intentional*; for the most part, it's not. But that doesn't make it any less misinforming.

So how can we acquire accurate knowledge of our noisy and complex world? Here are just a few suggestions.

- A good portion of human knowledge is very well established. Ground yourself in this stuff: math, computer science, [hard sciences](http://en.wikipedia.org/wiki/Hard_and_soft_science). Studying these subjects will give you a solid base of real knowledge to build from. Not only that, these fields of inquiry demonstrate careful reasoning and painstaking research, providing a start contrast with other fields.
- Realize that society is complex, and certain types of knowledge are difficult (sometimes impossible) to establish. Tone down your certainty, and keep an open mind. After all, there are smarter people than you who disagree with your views.
- Avoid "documentaries" and ideologues. (See below.)
- If you really want to stay apprised of the "news," at least use a variety of sources. Your one source may tell you what the "other side" believes. But in all likelihood, they're providing a caricature, or out-of-context quotes, or cherry-picking the weakest arguments. Make an honest effort to seek out the best thinkers and arguments that counter your views.

Humans are prone to a whole host of [well-documented cognitive mistakes](https://en.wikipedia.org/wiki/List_of_biases_in_judgment_and_decision_making) that make it hard to actually follow these suggestions. My *real* suggestion is: unlearn everything and restart from scratch, but that's a lot to ask.


### Ideologues

You get a pleasurable feeling of moral superiority each time you check your favorite ideological blogs or discuss politics with your like-minded peers. That pleasurably feeling is actually a burst of dopamine released in your brain, and it's damn addictive. Next time you're thinking about clicking over to your ideological blog, just picture a lab rat pushing its pedal for another shot of cocaine.

Over time, the addiction becomes stronger, and so does the (undue) level of certainty in your ideology. We've all met people who are so immersed in their ideology that they couldn't possibly change their minds. If you're on that path, break the cycle before it's too late.


### Code

For completion, here's the R code that generates the plots in the comic.

{% highlight r %}
set.seed(4)
n <- 50
X <- runif(n)
Y <- runif(n)
# There is no actual relationship between X and Y
plot(X, Y, main="What Happened this Week")

colA <- rep("black", n)
colA[which(abs(Y - X) < 0.2)] <- "red"
plot(X, Y, col=colA, main="What News Agency A Reported", col.main="red")

colB <- rep("black", n)
colB[which(abs(Y - 1 + X) < 0.2)] <- "blue"
plot(X, Y, col=colB, main="What News Agency B Reported", col.main="blue")
{% endhighlight %}
