Science-Wise False Discovery Rate
================
Nathan (Nat) Goodman
June 1, 2017

``` r
out=knitr::opts_knit$get("rmarkdown.pandoc.to");
paste(sep='','output format=',out);
```

    ## [1] "output format=markdown_github"

*A collection of R scripts exploring the statistical concept of science-wise false discovery rate (SWFDR) and illustrating software design choices in R. The main script is `swfdr_base.R`, a simple implementation that uses base R capabilities only. Other scripts extend the base implementation by providing solutions to some of the Exercises for the Reader.*

------------------------------------------------------------------------

`swfdr_base.R`
--------------

This script reimplements the core idea in [David Colquhoun's fascinating paper](http://rsos.royalsocietypublishing.org/content/1/3/140216), "An investigation of the false discovery rate and the misinterpretation of p-values" and further discussed in [Felix Schönbrodt's blog post](http://www.nicebread.de/whats-the-probability-that-a-significant-p-value-indicates-a-true-effect/), "What’s the probability that a significant p-value indicates a true effect?" and related [ShinyApp](http://shinyapps.org/apps/PPV/). The term *science-wise false discovery rate* is from [Jager and Leek's paper](http://doi.org/10.1093/biostatistics/kxt007), "An estimate of the science-wise false discovery rate and application to the top medical literature". [John Ioannidis’s landmark paper](http://dx.plos.org/10.1371/journal.pmed.0020124), “Why most published research findings are false”, is the origin of it all.

The *false discovery rate* (*FDR*) is the probability that a significant p-value indicates a false positive, or equivalently, the proportion of significant p-values that correspond to results without a real effect. The script produces graphs of FDR for a range of parameter values.

Installation and Usage
----------------------

The simplest way to get the software is to download the entire repository. The software is in the `script` subdirectory. At present, the only available script is `swfdr_base.R`. This program uses base R capabilities only and will run "out of the box" on any (reasonably modern) R installation.

The recommended way to run `swfdr_base.R` is to source the program into your R session and run the statement `run();` as shown below.

``` r
# this code block assumes your working directory is the root of the repository

source("script/swfdr_base.R");
run();
```

This runs the programs with default parameters produce four graphs similar to the ones below. The default process performs 2.5 × 10<sup>5</sup> simulations (taking about 3 minutes on my small Linux server).

<img src="../figure/swfdr_base/plot_byprop.png" width="50%" /><img src="../figure/swfdr_base/plot_byd.png" width="50%" /><img src="../figure/swfdr_base/plot_vsprop.png" width="50%" /><img src="../figure/swfdr_base/plot_vsd.png" width="50%" />

The notation is

-   solid lines show theoretical results; dashed lines are empirical results from the simulation
-   *fdr*. false discovery rate
-   *pval*. p-value cutoff for significance
-   *prop.true*. proportion of simulated cases that have a real effect
-   *d*. standardized effect size, aka *Cohen's d*

See Also
--------

These links edited by hand to explore how GitHub Pages handles relative links.

-   [Science: Science-Wise False Discovery Rate](doc/science.html) explains the scientific concepts underlying the software
-   [Software: Science-Wise False Discovery Rate](http://hddrugworks.org/ngoodman/NewPro/SWFDR/doc/software.html) discusses the software design and design choices
-   [Exercises: Science-Wise False Discovery Rate](http://hddrugworks.org/ngoodman/NewPro/SWFDR/doc/exercises.html) provides exercises for the reader to improve the program
-   There is also R-style [function-level documentation](TBD) and a [CHANGES file](http://hddrugworks.org/ngoodman/NewPro/SWFDR/CHANGES.html) that lists major differences between releases

HACK to test whether GitHub Pages will copy and display MDs. 

- [README.md](README.md)
- [doc/README.md](doc/README.md) copy of README
- [doc/foo.md](doc/foo.md) copy of README

HACK to test whether GitHub Pages will copy and display HTMLs. Also what happens when MD and HTML both present.

- [doc/foo.html](doc/foo.html) **edited** copy of README
- [doc/exercises.html](doc/exercises.html) empty file. title only

HACK to test whether GitHub Pages will copy and display PDFs. 

- [Science: Science-Wise False Discovery Rate (pdf)](doc/science.pdf)
- [Software: Science-Wise False Discovery Rate (pdf)](doc/software.pdf)

What happens to math?

- $x=y+z$
- $alpha=0.05$

The documentation is also in the `doc/` subdirectory.

Author
------

Nathan (Nat) Goodman, (natg at shore.net)

Bugs and Caveats
----------------

Please report any bugs, other problems, and feature requests using the [GitHub Issue Tracker](https://github.com/natgoodman/FDR/issues). I will be notified, and you'll be apprised of progress.

`swfdr_base.R` is pretty basic. See the [User Guide](http://hddrugworks.org/ngoodman/NewPro/SWFDR/doc/guide.html) and [Exercises](http://hddrugworks.org/ngoodman/NewPro/SWFDR/doc/exercises.html) for a list of known limitations, and other scripts in the `script/` subdirectory for solutions to some of these problems.

Copyright & License
-------------------

Copyright (c) 2017 Nathan Goodman

The software is **open source and free**, released under the [MIT License](https://opensource.org/licenses/MIT).
