---
title: ./experiment
date: 2011-02-7
layout: post
tags: science data reproducibility
---

Your computational biology experiment should be reproducible by typing

    ./experiment

## Hurdles

### Databases

Publicly-hosted databases such as GenBank change over time. How can one reproduce the 

## How to accomplish this

### Git

You should use version control to keep track of your scripts. I can't stress this enough. Combined with good readmes, Git has replaced my lab notebook.

### Rake/Make

I havne't used Make much but I have used the Ruby equivalent, [Rake](adsf) and it has changed my life. With Rake you can process dependencies. For example, if your pipeline requires a database from NCBI, Rake will first check to see that you have it and if not, download it:

{% highlight ruby %}
task :experiment => 'nr_rna.fasta'

file 'nr_rna.fasta' do
  sh "cat nt.fasta | tr 'T' 'U' > nt_rna.fasta"
end

file 'nr.fasta' do
   curl 'ftp://ftp.ncbi.nih.nlm.gov/...'
end

{% endhighlight %}

Enter this code into a file called `Rakefile` and type `Rake`

The first time, you will see the NCBI database being downloaded. Try running it again, nothing happens because all required files are present. Try deleting `nt_rna.fasta` before typing `Rake`. Now you will see Rake translating Ts into Us, without downloading the database from NCBI first (because it's already there).

### BASH

You can use bash to keep track of the parameters you use to run scripts
