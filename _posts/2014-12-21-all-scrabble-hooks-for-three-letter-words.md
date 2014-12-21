---
layout: post
title: "All Scrabble Hooks for Three letter Words"
category: tutorial
tags: [python]
---
{% include JB/setup %}


I'm a big Scrabble fan, and I was recently looking for a list of all the hooks for the three-letter words. For two letter words, this list ("the 2-to-make-3 list") is in the book *Everything Scrabble* and easy to find [on the web](http://www.wolfberg.net/scrabble/wordlists/OWL2/twos-to-threes.html). But I wasn't able to find an up-to-date ([OWL2](http://www.scrabbleplayers.org/w/Official_Tournament_and_Club_Word_List)) version of the 3-to-make-4 list. So I decided to make one for myself and share it here.

It's easy to find a list of all the allowed three-letter and four-letter words at sites such as [scrabutility](http://scrabutility.com/index.php). I've saved these word lists as text files [threes.txt](/static/threes.txt) and [fours.txt](/static/fours.txt). Then to create the list, I wrote some python code.

{% highlight py3 %}
# Read files to create list of three-letter words and list of four-letter words
def linestolist(filename):
	with open(filename) as f:
		return [x.strip('\n') for x in f.readlines()]
threes = linestolist('threes.txt')
fours = linetolist('fours.txt')

# For each three letter word, see which letters can go before it and after it
letters = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
before = ["" for x in range(len(threes))]
after = ["" for x in range(len(threes))]
for i in range(len(threes)):
	for l in letters:
		if l+threes[i] in fours:
			before[i] += l
		if threes[i]+l in fours:
			after[i] += l
z = zip(before, threes, after)

# Write the result as a CSV file
import csv
with open('threehooks.csv','w') as out:
    csv_out=csv.writer(out)
    csv_out.writerow(['before','word', 'after'])
    for row in z:
        csv_out.writerow(row)
{% endhighlight %}

I know the code isn't as computationally efficient as could be because it doesn't take into account the fact that the word files are sorted. But over-optimizing such a computationally small task isn't an efficient use of my time.


## The List

The list is is shown below; you can also download it as a [nicely-formatted pdf](/static/three-to-make-four.pdf).

| Front Hooks | Word | Back Hooks |
|:--------|:-------:|--------:|
| cell1   | cell2   | cell3   |
| cell4   | cell5   | cell6   |
{: class="table"}

