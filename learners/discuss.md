---
title: Discussion
---

## Parallel Execution

[SCons](https://scons.org/) can build dependencies in *parallel* sub-processes, via its `--jobs`
flag (or its `-j` abbreviation) which specifies the number of sub-processes to
use e.g.

```bash
$ scons --jobs=4 results.txt
```

If we have independent dependencies then these can be built at the
same time. For example, `abyss.dat` and `isles.dat` are mutually
independent and can both be built at the same time. Likewise for
`abyss.png` and `isles.png`. If you've got a bunch of independent
branches in your analysis, this can greatly speed up your build
process.

For more information see the SCons User Guide chapter on [Command-Line
Options](https://scons.org/doc/production/HTML/scons-user.html#sect-command-line-options).

## SCons and Version Control

Imagine that we manage our SCons configuration files using a version control system such as Git.

Let's say we'd like to run the workflow developed in this lesson
for three different word counting scripts, in order to compare their
speed (e.g. `wordcount.py`, `wordcount2.py`, `wordcount3.py`).

To do this we could edit `SConstruct` each time by replacing
`count_source="wordcount.py"` with `count_source="wordcount2.py"` or
`count_source="wordcount3.py"`,
but this would be detected as a change by the version control system.
This is a minor configuration change, rather than a change to the
workflow, and so we probably would rather avoid committing this change
to our repository each time we decide to test a different counting script.

An alternative is to leave `SConstruct` untouched, by overwriting the value
of `count_source` at the command line instead:

```
$ scons count_source=wordcount2.py
```

The configuration file then simply contains the default values for the workflow, and by overwriting
the defaults at the command line you can maintain a neater and more meaningful version control
history. Command line control of `SConstruct` files is documented in the SCons User Guide chapter on
[Command-Line
Options](https://scons.org/doc/production/HTML/scons-user.html#sect-command-line-options) and
[Command-Line Build
Variables](https://scons.org/doc/production/HTML/scons-user.html#sect-command-line-variables)

## SCons Variables and Shell Variables

`SConscript` files define shell commands, as the actions that are executed to update an object. More
complex actions could well include shell variables. There are several ways in which SCons special
substitution variables and shell variables can be confused and can be in conflict.

- SCons actually accepts two different syntaxes for action string substition variables: `$NAME` or or
  `${NAME}`.

  The `${NAME}` syntax is also used by the unix shell in cases where
  there might be ambiguity in interpreting variable names, or for
  certain pattern substitution operations.  Since there are only
  certain situations in which the unix shell requires this syntax,
  instead of the more common `$NAME`, it is not familiar to many users.

- SCons does variable substitution on actions before they are passed to
  the shell for execution. That means that anything that looks like a
  substitution variable to SCons will get replaced with the appropriate value. (In
  SCons, an uninitialized variable has an empty string value, `""`.) To protect a
  variable you intend to be interpreted by the shell rather than make,
  you need to "escape" the dollar sign by doubling it (`$$`). (This the
  same principle as escaping special characters in the unix shell
  using the backslash (`\`) character.) In
  short: SCons variables have a single dollar sign, shell variables
  have a double dollar sign. This applies to anything that looks like
  a variable and needs to be interpreted by the shell rather than
  SCons, including awk positional parameters (e.g., `awk '{print $$1}'`
  instead of `awk '{print $1}'`) or accessing environment variables
  (e.g., `$$HOME`).

::::::::::::::::::::::::::::::::::::::  discussion

## Detailed Example of Shell Variable Quoting

Say we had the following `SConstruct` file (and the .dat files had already
been created):

```python
books = "abyss isles"
plots = Command(
    target=[f"{book}.png" for book in books.split(" ")],
    source=[f"{book}.dat" for book in books.split(" ")],
    action=["for book in ${books}; do python plotcounts.py $book.dat $book.png; done"],
    books=books,
)
Alias("plots", plots)
```

the action that would be passed to the shell to execute would be:

```bash
for book in abyss isles; do python plotcounts.py ; done
```

Notice that SCons substituted `${books}`, as expected, but it also
substituted `$book`, even though we intended it to be a shell variable.
Moreover, because we didn't pass a `book` keyword argument, SCons
substituted an empty string '`""`' by default.

In order to get the desired behavior, we have to write `$$book` instead
of `$book`:

```python
books = "abyss isles"
plots = Command(
    target=[f"{book}.png" for book in books.split(" ")],
    source=[f"{book}.dat" for book in books.split(" ")],
    action=[for book in ${books}; do python plotcounts.py $$book.dat $$book.png; done"],
    books=books,
)
Alias("plots", plots)
```

which produces the correct shell command:

```bash
for book in abyss isles; do python plotcount.py $book.dat $book.png; done
```

::::::::::::::::::::::::::::::::::::::::::::::::::

## Build Managers and Reproducible Research

[Make](https://www.gnu.org/software/make/) is an older build manager than
[SCons](https://scons.org/). The original version of Make was released in 1977. Many build managers
are designed around the concepts originally introduced by Make. As the original build manager, it is
common to see Make appear in discussion about build managers.

Blog articles, papers, and tutorials on automating commonly
occurring research activities using Make:

- [minimal make][minimal-make] by Karl Broman. A minimal tutorial on
  using Make with R and LaTeX to automate data analysis, visualization
  and paper preparation. This page has links to Makefiles for many of
  his papers.

- [Why Use Make][why-use-make] by Mike Bostock. An example of using
  Make to download and convert data.

- [Makefiles for R/LaTeX projects][makefiles-for-r-latex] by Rob
  Hyndman. Another example of using Make with R and LaTeX.

- [GNU Make for Reproducible Data Analysis][make-reproducible-research]
  by Zachary Jones. Using Make with Python and LaTeX.

- Shaun Jackman's [Using Make to Increase Automation \&
  Reproducibility][increase-automation] video lesson, and accompanying
  [example][increase-automation-example].

- Lars Yencken's [Driving experiments with
  make][driving-experiments]. Using Make to sandbox Python
  dependencies and pull down data sets from Amazon S3.

- Askren MK, McAllister-Day TK, Koh N, Mestre Z, Dines JN, Korman BA,
  Melhorn SJ, Peterson DJ, Peverill M, Qin X, Rane SD, Reilly MA,
  Reiter MA, Sambrook KA, Woelfer KA, Grabowski TJ and Madhyastha TM
  (2016) [Using Make for Reproducible and Parallel Neuroimaging
  Workflow and
  Quality-Assurance][make-neuroscience]. Front. Neuroinform. 10:2. doi:
  10\.3389/fninf.2016.00002

- Li Haoyi's [What's in a Build Tool?][whats-a-build-tool] A review of
  popular build tools (including Make) in terms of their strengths and
  weaknesses for common build-related use cases in software
  development.

[gnu-make-parallel]: https://www.gnu.org/software/make/manual/html_node/Parallel.html
[makefile-variable]: https://stackoverflow.com/questions/448910/makefile-variable-assignment
[gnu-make-variables]: https://www.gnu.org/software/make/manual/html_node/Flavors.html#Flavors
[minimal-make]: https://kbroman.org/minimal_make/
[why-use-make]: https://bost.ocks.org/mike/make/
[makefiles-for-r-latex]: https://robjhyndman.com/hyndsight/makefiles/
[make-reproducible-research]: https://zmjones.com/make/
[increase-automation]: https://www.youtube.com/watch?v=_F5f0qi-aEc
[increase-automation-example]: https://github.com/sjackman/makefile-example
[driving-experiments]: https://lifesum.github.io/posts/2016/01/14/make-experiments/
[make-neuroscience]: https://journal.frontiersin.org/article/10.3389/fninf.2016.00002/full
[whats-a-build-tool]: https://www.lihaoyi.com/post/WhatsinaBuildTool.html
