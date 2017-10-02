Software: Science-Wise False Discovery Rate
================
Nathan (Nat) Goodman
June 1, 2017

*This document describes the software design for the program `swfdr_base.R` available in [my GitHub SWFDR repository](https://github.com/natgoodman/SWFDR). The companion document [Science: Science-Wise false Discovery Rate](%60science%60) explains the conceptual background. The [README file](../README.md) and [User Guide](%60guide%60) explain on how to get and run the program and interpret the results, and the R-style [function-level documentation](TBD) provide further details on the software.*

Introduction
------------

**Intro sentence or two TBD**

I hope the software will be a good example for readers who have some R experience but aren’t experts. The program uses base R capabilities only and will run "out of the box" on any reasonably modern R installation. It’s not a package but rather a collection of functions meant for interactive use, similar to what a non-expert R user is likely to write. It uses a simple, consistent coding style with lots of comments. The software (mostly) conforms to good software engineering practice: the code consists of small functions with well-defined purposes and clear interfaces (i.e., inputs and outputs).

I encourage reader-programmers to download the code, try it out, and modify it to do more. To this end, please see [Exercises: Science-Wise False Discovery Rate](exercises.md) for ideas on possible extensions. The software is open source, released under the MIT license, which means you can do almost anything you want with it. I’d love to see your improvements if you choose to share them, but you are under no obligation to do so.

Scenario
--------

The program simulates a large number of problem instances representing published results, some of which are true and some false. The instances are very simple: we generate two groups of random numbers and use the t-test to assess the difference between their means. One group (the control group or simply *group0*) comes from a standard normal distribution with mean=0. The other group (the treatment group or simply *group1*) is a little more involved:

-   for *true* instances, we take numbers from a standard normal distribution with mean *d* (d &gt; 0);
-   for *false* instances, we use the same distribution as group0.

The parameter *d* is the effect size, aka *Cohen’s d*.

We use the t-test to compare the means of the groups and produce a p-value assessing whether both groups comes from the same distribution.

We do this thousands of times (drawing different random numbers each time, of course), collect the resulting p-values, and compute the FDR. We repeat the procedure for a range of assumptions to determine the conditions under which most published results are wrong.

For *true* instances, we expect the difference in means to be approximately *d* and for *false* ones to be approximately 0, but due to the vagaries of random sampling, this may not be so. If the actual difference in means is far from the expected value, the t-test may get it wrong, declaring a *false* instance to be positive and a *true* one to be negative. The goal is to see how often we get the wrong answer across a range of assumptions.

Nomenclature
------------

To reduce confusion, I will be obsessively consistent in my terminology.

-   An *instance* is a single run of the simulation procedure.
-   The terms *positive* and *negative* refer to the results of the t-test. A *positive instance* is one for which the t-test reports a significant p-value; a *negative instance* is the opposite. Obviously the distinction between positive and negative rests on the chosen significance level.
-   *true* and *false* refer to the correct answers. A *true instance* is one where the treatment group (group1) is drawn from a distribution with mean=*d* (*d* ≠ 0). A *false instance* is the opposite: an instance drawn from a distribution with mean=0.
-   *empirical* refers to results calculated from the simulated data.

Software (swfdr\_base.R)
------------------------

The top-level function is `run`. The code block below shows how to invoke `run` with default parameters assuming your working directory is the root of the repository.

``` r
source("script/swfdr_base.R");
run();
```

`run` has 4 steps.

``` r
run=function(...) {
  init(...);                       # process parameters & other initialization
  doit();                          # do it! 
  saveit(save.rdata,save.txt);     # optionally save parameters and results
  fini();                          # final cleanup if any
}
```

-   `init` sets everything up
-   `doit` (pronounced “do it!”) does all the work
-   `saveit` (pronounced “save it!”) optionally saves the parameters, figures, and results
-   `fini` does final cleanup if any; there’s nothing to do in this example, but I include it for stylistic consistency.

A key consideration is this sort of program is how to pass parameters and results from one step to the next. I decided to use the very simple scheme of passing everything in global variables. This runs counter to conventional wisdom in the programming world but seemed okay provided it was well-documented. It’s also conducive for interactive use.

The table below lists the global variables. The first group are simulation parameters, next are parameters that control program operation, and last are results.

<table>
<colgroup>
<col width="19%" />
<col width="61%" />
<col width="19%" />
</colgroup>
<thead>
<tr class="header">
<th>variable</th>
<th>meaning</th>
<th>default or source</th>
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
<tr class="odd">
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>cases</td>
<td>parameter grid</td>
<td>created by <code>init</code></td>
</tr>
<tr class="odd">
<td>sim</td>
<td>simulation results</td>
<td>created by <code>dosim</code></td>
</tr>
<tr class="even">
<td>interp</td>
<td>interpolation results</td>
<td>created by <code>dointerp</code></td>
</tr>
</tbody>
</table>

`init` (code not shown) sets the parameters. Its argument list defines all parameters and provides default values. To change a parameter, simply call `run` (or `init` directly) with the parameter and new value, e.g., `run(n=32)` changes the sample size *n* from its default (16) to the new value (32). `init` creates a parameter grid, called *c**a**s**e**s*, containing all combinations of parameters. The code section at the bottom of `init` copies all parameters and its result, cases, into global variables.

For the default parameters, the parameter grid expands to 25 cases (5 values of *p**r**o**p*.*t**r**u**e* \* 5 values of *d*; all other parameters have single-valued defaults). We do 10,000 simulations for each case for a total of 250,000 simulations. This takes about 3 minutes on my small Linux server.

`doit` has 3 steps

``` r
doit=function() {
  dosim();                         # do simulation
  dointerp();                      # interpolate at fixed pvals
  doplot(save.plot);               # plot results and optionally save plots 
}
```

-   `dosim` runs the simulation
-   `dointerp` interpolates relevant columns of the simulation results at fixed p-values so we can plot results for p-values of interest
-   `doplot` plots the results and optionally save the plots.

`dosim` considers cases one by one, calling `sim_one` to do the heavy lifting, and stores the result in a global data frame, called *s**i**m*.

``` r
dosim=function() {
  ## call sim_one on each case and combine results into data frame
  ## the complicated one-liner below is a good idiom for doing this
  sim=do.call(rbind,apply(cases,1,
    function(case) do.call("sim_one",as.list(case)[c('prop.true','m','n','d')])));
  sim<<-sim;
  invisible(sim);
}
```

`sim_one` does the actual simulations. It draws *m* random samples of size *n* for each of the two groups, storing the numbers for each group in a *m* × *n* matrix whose rows are instances and columns are samples. The samples for group0 are from a standard normal distribution with mean=0, i.e. `rnorm(n,mean=0)`. For group1, the code computes the requisite numbers of true and false instances (*n**u**m*.*t**r**u**e* and *n**u**m*.*f**a**l**s**e*, resp.), draws *n**u**m*.*t**r**u**e* samples from a standard normal distribution with mean=d, i.e. `rnorm(n,mean=d)` and *n**u**m*.*f**a**l**s**e* samples from the same distribution as group0.

The program runs the t-test on each of the *m* pairs of samples, generating *m* p-values, and calculates theoretical and empirical FDRs for each p-value. It also calculates and stores basic statistics for each pair of samples, e.g., the actual means and differences, should this be of interest later.

``` r
sim_one=function(prop.true,m,n,d) {
  ## draw m pairs of samples
  ## each group is matrix whose rows are cases and columns are samples
  ## group0 is control
  group0=replicate(m,rnorm(n,mean=0));
  ## group1 contains num.false samples with effect=0, and num.true with effect=1
  num.true=round(m*prop.true);
  num.false=m-num.true;
  ## be careful when num.false or num.true is 0: replicate(0,...) produces empty list!
  if (num.false==0) group10=NULL else group10=replicate(num.false,rnorm(n,mean=0));
  if (num.true==0) group11=NULL else group11=replicate(num.true,rnorm(n,mean=d));
  group1=cbind(group10,group11);
  ## set vector of true effects
  d.true=c(rep(FALSE,num.false),rep(TRUE,num.true));
  ## apply t-test to all m pairs of samples
  pval=sapply(1:m,function(j) t.test(group0[,j],group1[,j])$p.value);
  ## get means and standard deviations of samples and differences between means
  mean0=colMeans(group0);
  mean1=colMeans(group1);
  sd0=sapply(1:m,function(j) sd(group0[,j]));
  sd1=sapply(1:m,function(j) sd(group1[,j]));
  diff=mean1-mean0;
  ## calculate theoretical and simulated (empirical) fdr
  fdr.theo=fdr_theo(pval,num.true,num.false,n,d);
  fdr.empi=fdr_empi(pval,d.true);
  ## store it all in data frame
  sim=data.frame(prop.true,m,n,d,d.true,pval,mean0,mean1,diff,sd0,sd1,
    fdr.theo=fdr.theo,fdr.empi=fdr.empi);
  invisible(sim);
}
```

`sim_one` calls functions `fdr_theo` and `fdr_empi` to do the FDR calculations. Recall that FDR is the number of false positives divided by the total number of positives, and of course, the total number of positives is the number of false positives plus the number of true ones. In pseudo-math: *F**D**R* = *n**u**m*.*f**p*/(*n**u**m*.*f**p* + *n**u**m*.*t**p*).

The theoretical FDR uses textbook formulas to estimate *n**u**m*.*f**p* and *n**u**m*.*t**p* for each p-value *p**v**a**l*:

-   *n**u**m*.*f**p* = *p**v**a**l* times the number of false instances = *p**v**a**l* × (1 − *p**r**o**p*.*t**r**u**e*)×*m*
-   *n**u**m*.*t**p* = power (given *p**v**a**l*) times the number of true instances = *p**o**w**e**r* × *p**r**o**p*.*t**r**u**e* × *m*

The program calculates power using R’s power.t.test function: `power.t.test(n=n,delta=d,sd=1,sig.level=p)$power`.

The empirical FDR is conceptually simpler: for each p-value *p**v**a**l*, the code counts the number of false positives in instances with p-values ≤*p**v**a**l* and divides by the number of all instances with such p-values.

The structure of `dointerp` is similar. It calls `interp_one` on each case to interpolate the theoretical and empirical FDRs at desired p-values using R’s `approxfun` function. `dointerp` stores the result in a global data frame, called interp.

``` r
dointerp=function() {
  ## call interp_one on each case and combine results into data frame
  ## the complicated one-liner is a good idiom for doing this
  interp=do.call(rbind,apply(cases,1,
    function(case) do.call("interp_one",as.list(case)[c('prop.true','m','n','d')])));
  interp<<-interp;
  invisible(interp);
}
interp_one=function(prop.true,m,n,d) {
  sim1=sim[(sim$prop.true==prop.true&sim$m==m&sim$n==n &sim$d==d),];
  fun_theo=with(sim1,approxfun(pval,fdr.theo,rule=2));
  fdr.theo=fun_theo(pval.interp);
  fun_empi=with(sim1,approxfun(pval,fdr.empi,rule=2));
  fdr.empi=fun_empi(pval.interp);
  interp=data.frame(prop.true,m,n,d,pval=pval.interp,fdr.theo,fdr.empi);
  invisible(interp);
}
```

`doplot` operates on the *s**i**m* and *i**n**t**e**r**p* data frames. It calls separate functions for each of the four kinds of plot.

``` r
doplot=function(save.plot=F) {
    plot_byprop(save.plot);             # plot fdr by prop.true
    plot_byd(save.plot);                # plot fdr by d
    plot_vsprop(save.plot);             # plot fdr for fixed pvals vs prop.true
    plot_vsd(save.plot);                # plot fdr for fixed pvals vs d
}
```

-   `plot_vsd` plots FDR for fixed p-values vs effect size (*d*)
-   `plot_vsprop` plots FDR for fixed p-values vs *p**r**o**p*.*t**r**u**e*
-   `plot_byd` plots FDR by effect size (*d*)
-   `plot_byprop` plots FDR by *p**r**o**p*.*t**r**u**e*

`saveit` saves all parameters and results (actually, all global variables) if the user so wishes; a companion function, `loadit`, loads the saved data into the R session.

``` r
saveit=function(save.rdata=T,save.txt=F) {
  if (save.rdata) {
    ## get names of global variables that aren't functions
    vars=Filter(function(x) !is.function(get(x,envir=.GlobalEnv)),ls(envir=.GlobalEnv));
    save(list=vars,envir=.GlobalEnv,file=file.path(datadir,'globals.RData'));
  }
  if (save.txt) {
    write.table(sim,file=file.path(datadir,'sim.txt'),sep='\t',quote=F,row.names=F);
    write.table(interp,file=file.path(datadir,'interp.txt'),sep='\t',quote=F,row.names=F);
  }
}
loadit=function(file='data/swfdr_base/globals.RData') {
  if (!file.exists(file))
    stop(paste(sep=' ',"Cannot load saved parameters and results: file",file,"does not exist"));
  load(file=file,envir=.GlobalEnv);
}
```

Programming Style
-----------------

You may have noticed that my R code looks different from other examples out there and eschews many recommendations in Hadley Wickham's and Google's R style guides. One obvious difference is that my code is scrunched with less whitespace than others. In addition, I use `=` instead of `<-`, prefer single-quotes over double, end statements with semi-colons, start full line comments with \#\#, and use `.` as a word separator in variable names and `_` in function names. I have good reasons for some of these choices, but mostly it’s habit.

Programming style is a matter of taste (unless your boss foists a style on you). My style works for me. As you gain experience, I urge you to find a style that works for you.

**Conclusion or Wrap-up TBD**
