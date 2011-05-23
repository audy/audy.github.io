---
title: Ghetto BLAST
date: 2011-05-23
layout: post
comments: true
---
## Searching DNA databases using Zlib

I read an article somewhere that described a simple data mining technique that used compression to search a data set for strings _similar_ to a given query string. I can't find that article now but I always wanted to try it. The method works by comparing the size of the compressed versions of the strings to the size of the strings if they are compressed together.

For example, if you want to compare two strings to see if they are similar, first compress them separately and get the length of the compressed strings:

{% highlight ruby %}

>> compress("Austin")
=> 9
>> compress("Boston")
=> 9

{% endhighlight %}

Now compress them together:

{% highlight ruby %}

>> compress("AustinBoston")
=> 12
		
{% endhighlight %}

12 is still more than 9 but less than 18 (the sum of separately compressed sizes). So the strings must be similar.

I wanted to see if this would work with querying DNA databases for similar genes, so I wrote a [little script](https://gist.github.com/987717).

I did some quick testing on a 16S database and by golly it seems to work. Even with partial sequences. I would like to do some benchmarking to see how accurate it is. 

This is by far not the most efficient approach to solving this problem but it is pretty clever.