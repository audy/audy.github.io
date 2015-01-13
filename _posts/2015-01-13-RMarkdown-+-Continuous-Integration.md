---
title: RMarkdown & Continuous Integration
layout: post
---

I added continuous integration using [Travis-CI](https://travis-ci.com) to my
[Make + RMarkdown example](https://github.com/audy/make-rmarkdown). When GitHub
receives a `git push`, Travis will clone the repository and run `make` which
will build all RMarkdown files and notify you of any failures.

This could also be used to build RMarkdown files automatically if the output
could somehow be sent elsewhere. One way to do this would be to send them to a
remote machine using secure-copy or FTP.

One thing this configuration does not currently handle is downloading and
installing dependencies. R's really only solution for package management is
[Packrat](https://rstudio.github.io/packrat/) which requires you to check in and
track all dependencies in Git. I might experiment with adding this.

See [`.travis.yml`](https://github.com/audy/make-rmarkdown/blob/master/.travis.yml)
