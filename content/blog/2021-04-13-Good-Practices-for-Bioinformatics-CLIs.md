+++
title = "Good Practices for Bioinformatics CLIs"
date = 2021-04-13
description = "Some opinions on common bioinformatics software patterns"
+++

# Scenario

You're the author of an important bioinformatics program that can accurately
predict some biologically-meaningful properties. The scientific community would
like to use your program as part of a larger pipeline. However, some common
follies / design decisions in bioinformatics software can make that job
difficult:

1. Running External Programs
2. Too Much Magic #1: Automatic Filename(s)
3. Too Much Magic #2: Automatic Updates

# Running External Programs

This section is dedicated to the countless hours I have spent debugging tools
which execute an external process but do not produce an error if that external
process fails for some reason.

## Avoid `os.system`

`os.system` is the shortest to type method for running an external program but
commonly causes bugs due to the fact that if the command fails, the Python
program will continue to execute as if nothing happened, sometimes without
noticing.

This example shows what happens if you run a command that fails with
`os.system`

```python
import os

print("generating myfile.txt using nonsense.exe")

exit_code = os.system("nonsense.exe --output myfile.txt")

print("finished generating myfile.txt")

assert exit_code == 0, "You *can* make sure that the program succeeded, but people often forget"
```

Any downstream code that assumes `myfile.txt` exists will now break. Also,
because `os.system` did not raise an exception, it becomes more difficult to
find out the reason why `myfile.txt` does not exist.

`subprocess.call` is a little better because it will fail if `nonsense.exe`
does not exist. However, if you try to run a real command but it fails, the
script continues as if nothing bad happened:


```python
import subprocess
# the script would crash here
subprocess.call(["nonsense", "output.txt"])
```

```python
exit_code = subprocess.call(["ls", "--not-a-real-argument"])

print("finished running ls")

assert exit_code == 0, "You *can* make sure that the program succeeded, but people often forget"
```

If you really wanted to make this safe, you'd have to do the following:

```python
try:
    exit_code = subprocess.call(["ls", "--not-a-real-argument"])
except FileNotFoundError:
    print("program does not exist")
finally:
    assert exit_code == 0, "Program did not exit successfully"
```

But that involves a lot more typing than simply using `subprocess.check_output`:

```python
# works
subprocess.call(["ls"])

# FileNotFoundError
subprocess.check_output(["notacommand"])

# subprocess.CalledProcessError
subprocess.call(["ls", "--not-a-real-argument"])
```

*tl;dr* It's safest and easiest to just use `subprocess.check_output` for
running external commands. For more advanced features such as reading from
standard out or standard error, see `subprocess.Popen`.

# Avoid: Automatically Generated Output Filenames

Many tools I've encountered will automatically generate the paths to output
file(s). This is done to help the user. Who wants to manually specify the
output filename especially if there are more than one? This usually causes
issues down the line when a user tries to run a program using another program
as it adds work to guess the path to the output file(s).

For example, many tools use the input filename as the prefix to all of the
outputs.

```sh
mytool --input sample123.fastq
ls
# sample123.fastq sample123.results.csv
```

Let's say the user now wants to count how many lines there are in the output.
They now have to write additional code to generate the output file path

```sh
input_filename=sample123.fastq

mytool --input sample123.fastq

results_path=$(basename ${input_filename} .fastq).results.csv
```

Consider a user wrapping this tool in a script:

```sh
input_filename="sample123.fastq"
mytool --input "${input_filename}"
ls

# sample123.fastq.results-20210513.csv
```

Let's say the user now wants to count how many lines there are in the output.
The code needed to figure out the output path starts to become more complicated
and brittle due to `mytool` and the wrapping script having to implement
formatting the date in the exact same way:

```sh
input_filename="sample123.fastq"

mytool --input "${input_filename}"

output_path="sample123.fastq.results-$(date +%Y%m%d).csv"
wc -l "${output_path}"
```

If you allow the user to provide the output path(s) as arguments, the script
becomes much simpler:

```sh
input_filename=sample123.fastq

mytool --input sample123.fastq --output results.csv

wc -l results.csv
```

*tl;dr* while it might appear that automatically generating output filenames
makes a user's life easier; it ends up adding work down the road.

# Avoid: Automatically Self-Updating Code / Data

Tools that use an external database sometimes check to see if the local copy of
that database is up-to-date and will automatically check for and download a
newer version from the internet if available.

While it is often important to have an up-to-date copy of a database this
breaks reproducibility and can lead to failures if the server hosting the
database goes offline. When creating robust bioinformatics tools, being able to
produce a result reproducibly is more important than unintentially changing the
output because an external database has been updated.

```sh
# bad
$ mytool --input sample123.fastq
# mytool: checking for updates
# mytool: found newer database... downloading

sample123.fastq.results.csv # <- no longer reproducible result
```

As an alternative, I recommend not automatically updating and instead provide a
command to update the database. Then I have control over when the database is
updated and can keep copies of multiple versions if necessary:

```sh
mytool update-database --output database.fasta

mytool run-analysis --input sample123.fastq --database database.fasta
```
