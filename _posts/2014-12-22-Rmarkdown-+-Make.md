---
title: Rmarkdown + Make
date: 2014-12-22
tags: R science Make programming reproducibility
layout: post
---

Lately, I've been using Rmarkdown + Make to perform statistical analysis and
generate reports.

I saw Shaun Jackman's slideshow "[Open, reproducible science using Make,
RMarkdown and Pandoc][1]" and thought it was lacking a key example: how do you
use Make to generate reports from RMarkdown files? So I put together a [little
recipe][2] for doing just that. It is a basic project directory structure and a
tiny Makefile that automatically renders Rmarkdown files to HTML.

For example:

```
$ make
notebooks/1-hello-world.Rmd -> notebooks/1-hello-world.html
echo "library(knitr); knit(\"notebooks/1-hello-world.Rmd\", \"notebooks/1-hello-world.html\")" | R --slave --vanilla --no-save


processing file: notebooks/1-hello-world.Rmd
  |.................................................................| 100%
  ordinary text without R code


output file: notebooks/1-hello-world.html

[1] "notebooks/1-hello-world.html"
```

[1]: https://sjackman.github.io/open-science/#/open-reproducible-science
[2]: https://github.com/audy/knitr-make
