User Guide: Science-Wise False Discovery Rate
================
Nathan (Nat) Goodman
June 1, 2017

*This user guide explains how to install and run the scripts in my [SWFDR GitHub repository](https://github.com/natgoodman/SWFDR). The main script is `swfdr_base.R`, a simple implementation that uses base R capabilities only. Other scripts extend the base implementation by providing solutions to some of the Exercises for the Reader.*

------------------------------------------------------------------------

`swfdr_base.R`
--------------

This script reimplements the core idea in [David Colquhoun's fascinating paper](http://rsos.royalsocietypublishing.org/content/1/3/140216), "An investigation of the false discovery rate and the misinterpretation of p-values" and further discussed in [Felix Schönbrodt's blog post](http://www.nicebread.de/whats-the-probability-that-a-significant-p-value-indicates-a-true-effect/), "What’s the probability that a significant p-value indicates a true effect?" and related [ShinyApp](http://shinyapps.org/apps/PPV/). The term *science-wise false discovery rate* is from [Jager and Leek's paper](http://doi.org/10.1093/biostatistics/kxt007), "An estimate of the science-wise false discovery rate and application to the top medical literature". [John Ioannidis’s landmark paper](http://dx.plos.org/10.1371/journal.pmed.0020124), “Why most published research findings are false”, is the origin of it all.

The *false discovery rate* (*FDR*) is the probability that a significant p-value indicates a false positive, or equivalently, the proportion of significant p-values that correspond to results without a real effect. The complement, *positive predictive value (PPV=1-FDR)* is the probability that a significant p-value indicates a true positive, or equivalently, the proportion of significant p-values that correspond to results with real effects.

The script produces graphs of FDR for a range of parameter values. The user can easily change parameters and rerun the program.

### Installation and Usage

The simplest way to get the software is to download the entire repository. The software is in the `script` subdirectory. At present, the only available script is `swfdr_base.R`. This program uses base R capabilities only and will run "out of the box" on any (reasonably modern) R installation.

The recommended way to run `swfdr_base.R` is to source the program into your R session and run the statement `run();` The code below illustrates the default process and some variants.

``` r
# this code block assumes your working directory is the root of the repository

source("script/swfdr_base.R");
# run default process
run();

# run default process and save results in directories data/guide01 and figure/guide01
run(save=T,datadir='data/guide01,figdir='figure/guide01);

# reduce runtime by reducing number of simulation runs and simulated cases
run(m=1e3,d=c(0.25,0.5,1),prop.true=c(0.3,0.5,0.8));

# specify power directly and let program adjust effect size
run(m=1e3,pwr=c(0.1,0.3,0.8),prop.true=c(0.3,0.5,0.8));

# skip computation by loading saved results and replotting graphs
loadit();
doplot();

# load saved results and do something completely different
# for example, plot distribution of effect size error for small effects with significant p-values 
loadit();
with(subset(sim,subset=(d==0.25&d.true&pval<=0.05)),hist(diff-d));
```

The default process performs 2.5 × 10<sup>5</sup> simulations (taking about 3 minutes on my small Linux server) and produce four graphs similar to the ones below (in directory `figure/swfdr_base`). The other examples are much faster. The plots for the three `run` examples and the final histogram example are in directories `figure/guide01`, etc. The plots for the `loadit(); doplot();` example are identical to the default process.

<img src="../figure/swfdr_base/plot_byprop.png" width="50%" /><img src="../figure/swfdr_base/plot_byd.png" width="50%" /><img src="../figure/swfdr_base/plot_vsprop.png" width="50%" /><img src="../figure/swfdr_base/plot_vsd.png" width="50%" />

The nomenclature is

-   solid lines show theoretical results; dashed lines are empirical results from the simulation
-   *f**d**r*. false discovery rate
-   *p**v**a**l*. p-value cutoff for significance
-   *p**r**o**p*.*t**r**u**e*. proportion of simulated cases that have a real effect
-   *d*. standardized effect size, aka *Cohen's d*

The user can change simulation parameters and control program operation by providing new values to `run()` as illustrated in the example code block above. The available parameters are

<table>
<colgroup>
<col width="19%" />
<col width="61%" />
<col width="19%" />
</colgroup>
<thead>
<tr class="header">
<th>parameter</th>
<th>meaning</th>
<th>default</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>prop.true</td>
<td>fraction of cases where there is a real effect</td>
<td><code>seq(.1,.9,by=.2)</code></td>
</tr>
<tr class="even">
<td>m</td>
<td>number of iterations</td>
<td><code>1e4</code></td>
</tr>
<tr class="odd">
<td>n</td>
<td>sample size</td>
<td><code>16</code></td>
</tr>
<tr class="even">
<td>d</td>
<td>standardized effect size (aka Cohen's d)</td>
<td><code>c(.25,.50,.75,1,2)</code></td>
</tr>
<tr class="odd">
<td>pwr</td>
<td>power. if set, program adjusts <span class="math inline"><em>d</em></span> to achieve power</td>
<td><code>NA</code></td>
</tr>
<tr class="even">
<td>sig.level</td>
<td>significance level for power calculations when <span class="math inline"><em>p</em><em>w</em><em>r</em></span> is set</td>
<td><code>0.05</code></td>
</tr>
<tr class="odd">
<td>pval.interp</td>
<td>p-values for which we plot results</td>
<td><code>c(.001,.01,.03,.05,.1)</code></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>scriptname</td>
<td>used to set output directories and in error messages</td>
<td><code>'swfdr_base'</code></td>
</tr>
<tr class="even">
<td>datadir</td>
<td>data directory relative to distribution root</td>
<td><code>'data/swfdr_base'</code></td>
</tr>
<tr class="odd">
<td>figdir</td>
<td>figure directory relative to distribution root</td>
<td><code>'figure/swfdr_base'</code></td>
</tr>
<tr class="even">
<td>save</td>
<td>save parameters, results, and plots; sets <span class="math inline"><em>s</em><em>a</em><em>v</em><em>e</em>.<em>r</em><em>d</em><em>a</em><em>t</em><em>a</em></span> and <span class="math inline"><em>s</em><em>a</em><em>v</em><em>e</em>.<em>p</em><em>l</em><em>o</em><em>t</em></span></td>
<td><code>FALSE</code></td>
</tr>
<tr class="odd">
<td>save.rdata</td>
<td>save parameters and results in RData format</td>
<td><code>FALSE</code> (set by <span class="math inline"><em>s</em><em>a</em><em>v</em><em>e</em></span>)</td>
</tr>
<tr class="even">
<td>save.txt</td>
<td>save results in txt format. <strong>CAUTION</strong>: big &amp; slow</td>
<td><code>FALSE</code> (not set by <span class="math inline"><em>s</em><em>a</em><em>v</em><em>e</em></span>)</td>
</tr>
<tr class="odd">
<td>save.plot</td>
<td>save plots</td>
<td><code>FALSE</code> (set by <span class="math inline"><em>s</em><em>a</em><em>v</em><em>e</em></span>)</td>
</tr>
<tr class="even">
<td>clean</td>
<td>remove contents of data and figure directories and start fresh; sets <span class="math inline"><em>c</em><em>l</em><em>e</em><em>a</em><em>n</em>.<em>d</em><em>a</em><em>t</em><em>a</em></span> and <span class="math inline"><em>c</em><em>l</em><em>e</em><em>a</em><em>n</em>.<em>f</em><em>i</em><em>g</em></span></td>
<td><code>FALSE</code></td>
</tr>
<tr class="odd">
<td>clean.data</td>
<td>remove contents of data directory</td>
<td><code>FALSE</code> (set by <span class="math inline"><em>c</em><em>l</em><em>e</em><em>a</em><em>n</em></span>)</td>
</tr>
<tr class="even">
<td>clean.fig</td>
<td>remove contents of figure directory</td>
<td><code>FALSE</code> (set by <span class="math inline"><em>c</em><em>l</em><em>e</em><em>a</em><em>n</em></span>)</td>
</tr>
</tbody>
</table>

### Statistical Details

-   The program assumes normally distributed data with equal variance.
-   The p-values are from a two-sided, unpaired, equal variance t-test (R's `t.test` with default settings for `alternative`, `paired`, and `var.equal`).
-   The default sample size (*n*=16) gives about 80% power for *d* = 1 and *s**i**g*.*l**e**v**e**l* = 0.05. Power for the full range of *d* is

| *d*   | 0.25 | 0.50 | 0.75 | 1.00 |  2.00  |
|-------|------|------|------|------|:------:|
| power | 0.10 | 0.28 | 0.54 | 0.78 | 0.9998 |

Directory Structure
-------------------

The root directory contains the usual GitHub files: LICENSE, README, .gitignore. There's also a small R file `fixlinks.R` used to fix links in the documentation. The subdirectories are

-   `data/` - data files
-   `doc/` - documentation
-   `figure/` - plots
-   `script/` - scripts

Other Scripts
-------------

TBD

Documentation
-------------

The documentation comprises

-   this User Guide
-   the [README file](../README.md) in the root directory
-   R-style [function-level documentation](TBD)
-   [Science: Science-Wise False Discovery Rate](science.md) explains the scientific concepts underlying the software
-   [Software: Science-Wise False Discovery Rate](software.md) describes the software design
-   [Exercises: Science-Wise False Discovery Rate](exercises.md) provides exercises for the reader to improve the program.

Each document has four files.

1.  R markdown (`Rmd`) is the source file: I generate the others programmatically using, e.g.,

        rmarkdown::render(guide.Rmd(c('github_document','html_document','pdf_document'))

2.  markdown (`md`) is the format GitHub prefers for documents displayed on its site.
3.  `html`. These are self-contained HTML files that should display correctly on your local computer or website. Because they are self-contained, they may be huge and inscrutable. Note that GitHub shows HTML files as raw text, which is not very illuminating, but has an [HTML preview feature](http://htmlpreview.github.io) that renders the files as expected.
4.  `pdf`. GitHub renders the files as expected **but ignores links**.

See Also
--------

Felix Schönbrodt's [blog post](http://www.nicebread.de/whats-the-probability-that-a-significant-p-value-indicates-a-true-effect/) and [ShinyApp](http://shinyapps.org/apps/PPV/) got me going down this path and offer an insightful, different perspective. Blog posts by [Daniel Lakens](http://daniellakens.blogspot.de/2015/09/how-can-p-005-lead-to-wrong-conclusions.html) and [Will Gervais](http://willgervais.com/blog/2014/9/24/power-consequences%3E) are also interesting.

Papers by [David Colquhoun](http://rsos.royalsocietypublishing.org/content/1/3/140216), [Leah Jager and Jeffrey Leek](http://doi.org/10.1093/biostatistics/kxt007), and [John Ioannidis](http://dx.plos.org/10.1371/journal.pmed.0020124) cover it all with full statistical rigor and much more detail.

Author
------

Nathan (Nat) Goodman, (natg at shore.net)

Bugs and Caveats
----------------

Please report any bugs, other problems, and feature requests using the [GitHub Issue Tracker](https://github.com/natgoodman/FDR/issues). I will be notified, and you'll be apprised of progress.

### Known Bugs and Caveats

-   The simulation results are noisy for *m* &lt; 5x10<sup>3</sup> and dubious for *m* &lt; 10<sup>3</sup>; the program does not run at all for *m* &lt; 2.
-   The software is pretty basic. See [Exercises: Science-Wise False Discovery Rate](exercises.md) for a list of known limitations.

Copyright & License
-------------------

Copyright (c) 2017 Nathan Goodman

The software is **open source and free**, released under the [MIT License](https://opensource.org/licenses/MIT).
