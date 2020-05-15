---
title: Polyglot Makefiles
layout: post
---

Did you know that you can change the shell in a `Makefile`? It's true. I found
this out when trying to use bash instead of the default `/bin/sh` by setting
`SHELL`.

```makefile
.ONESHELL:
bash: SHELL := bash
bash:
  # note: the double dollar-sign is required because Make substitues $variables
	export greeting="¡hola"
	echo "$${greeting}, bash!"
```

What else is possible? Can I use other programming languages as the shell?  Is
it possible to write in-line Python in a `Makefile`?

```makefile
.ONESHELL:
python: SHELL := python3
python:
	greeting = "hello"
	print(f"{greeting}, python!")
```

Typing `make python` will print `hello, python!`. Notice that there is now a
new directive, `.ONESHELL`. Normally, `make` will evaluate each command in a
separate shell meaning that, in this example, `greeting` would be undefined in
the second line. Adding `.ONESHELL` to the top of your `Makefile`, as
recommended by someone else whose last name begins with Davis-\* and blogs
about `make`, in ["Your Makefiles are wrong"](https://tech.davis-hansson.com/p/make/).

What about writing R in-line in a Makefile? Note the addition of `.SHELLFLAGS`.
By default, `make` runs `SHELL -c "your\nscript\nhere"` which is not compatible
R. To run a script in-line from a command in R, you use `-e`:

```makefile
R: .SHELLFLAGS := -e
R: SHELL := Rscript
R:
	greeting = "bonjour"
	message(paste0(greeting, ", R!"))
```

**sigh** changing the interpreter is boring to me now. I want to write scripts
in-line in a `Makefile` that run in Docker containers. Yes, that's possible
too:

```makefile
docker: .SHELLFLAGS = run --rm --entrypoint /bin/bash ubuntu -c
docker: SHELL := docker
docker:
	echo "hello, $$(uname -a)!"
```

This prints:

```
hello, Linux 453c728113d6 4.19.76-linuxkit #1 SMP Fri Apr 3 15:53:26 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux!
```

## Why?

This started as an experiment to see what is possible with `make`. But it was
motivated by a real world problem: I often need to combine tools across
programming languages and environments to run data analysis pipelines. I like
to automate these processes and make is a great tool for doing so. Having
everything in-line is more readable and having everything in a single file
means that I have to jump between fewer tabs to understand the pipeline itself.

## Full Makefile

**note**: Some of these features only work on `make 4.+` which I installed
using [Homebrew](https://brew.sh). These examples do not work using `make 3.x`
which is comes with macOS Catalina.

```makefile
.ONESHELL:
.SILENT:

main: \
	python \
	ruby \
	R \
	bash \
	docker

docker: .SHELLFLAGS = run --rm --entrypoint /bin/bash ubuntu -c
docker: SHELL := docker
docker:
	echo "hello, $$(uname -a)!"

python: SHELL := python3
python:
	greeting = "hello"
	print(f"{greeting}, python!")

ruby: .SHELLFLAGS := -e
ruby: SHELL := ruby
ruby:
	greeting = "labas"
	puts "#{greeting}, ruby!"

R: .SHELLFLAGS := -e
R: SHELL := Rscript
R:
	greeting = "bonjour"
	message(paste0(greeting, ", R!"))

bash: .SHELLFLAGS := -euo pipefail -c
bash: SHELL := bash
bash:
	export greeting="¡hola"
	echo "$${greeting}, bash!"
```
