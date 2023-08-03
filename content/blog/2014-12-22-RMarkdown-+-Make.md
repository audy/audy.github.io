+++
title = "RMarkdown + Make"
date = 2014-12-22
+++

Lately, I've been using Rmarkdown + Make to perform statistical analysis and
generate reports.

I saw Shaun Jackman's slideshow "[Open, reproducible science using Make,
RMarkdown and Pandoc][1]" and thought it was lacking a key example: how do you
use Make to generate reports from RMarkdown files? So I put together a [little
recipe][2] for doing just that. It is a basic project directory structure and a
tiny Makefile that automatically renders Rmarkdown files to HTML.

In the example repo, there is a `Makefile` which defines a couple of rules for
converting `.md` files into `.pdf` and `.html` files:

```makefile
%.html: %.Rmd
	@Rscript -e "rmarkdown::render('$<')"

%.pdf: %.Rmd
	@Rscript -e "rmarkdown::render('$<')"
```

After adding this to your `Makefile` you can type `make my-analysis.pdf` to
render `my-analysis.md`.

To automatically convert all `.md` files to `.pdf`, add the following lines
which will create a target `.pdf` file for all of the `.Rmd` files in the
current working directory:

```makefile
# define targets
SOURCES=$(shell find notebooks -name *.Rmd)
TARGETS=$(SOURCES:%.Rmd=%.pdf)

default: $(TARGETS)

# type `make clean` to delete all of the target PDFs
clean:
	rm -rf $(TARGETS)
```


[1]: https://sjackman.github.io/open-science/#/open-reproducible-science
[2]: https://github.com/audy/knitr-make
