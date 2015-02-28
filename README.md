# investr: Inverse Estimation in R

Inverse estimation, also referred to as the calibration problem, is a classical and well-known problem in regression. In simple terms, it involves the use of an observed value of the response (or specified value of the mean response) to make inference on the corresponding unknown value of the explanatory variable. 

A detailed introduction to investr has been published in The R Journal: "investr: An R Package for Inverse Estimation", http://journal.r-project.org/archive/2014-1/greenwell-kabban.pdf. You can track development at https://github.com/w108bmg/investr. To report bugs or issues, contact the main author directly or submit them to https://github.com/w108bmg/investr/issues. 

As of right now, `investr` supports (univariate) inverse estimation with objects of class:

* `lm` --- linear models
* `glm` --- generalized linear models
* `nls` --- nonlinear least-squares models
* `lme` --- linear mixed-effects models (fit using the `nlme` package)

## Installation
The package is currently listed on CRAN and can easily be installed:
```r
  ## Install from CRAN
  install.packages("investr", dep = TRUE)
```
The package is also part of the [ChemPhys task view](http://cran.r-project.org/web/views/ChemPhys.html) --- a collection of R packages useful for analyzing data from chemistry and physics experiments. These packages can all be installed at once (including `investr`) using the `ctv` package (Zeileis, 2005):
```r
  ## Install the ChemPhys task view
  install.packages("ctv")
  ctv::install.views("ChemPhys")
```

## Examples

### Estimating the 50% lethal dose in binomial regression

In binomial regression, the estimated lethal dose corresponding to a specifi probability $p$ of death is often referred to as $LD_p$. `invest` obtains an estimate of $LD_p$ by inverting the fitted mean response on the link scale. Similarly, by default, a confidence interval for $LD_p$ is obtained by inverting a confidence interval for the response on the link scale.
```r
library(investr)

## Dobson's beetle data
head(beetle)

## Binomial regression
binom_fit <- glm(cbind(y, n-y) ~ ldose, data = beetle, 
                 family = binomial(link = "cloglog"))
plotFit(binom_fit, lwd.fit = 2, cex = 1.2, pch = 21, bg = "lightskyblue", 
        lwd = 2)

## Estimate the 50% lethal dose
invest(binom_fit, y0 = 0.5)

# estimate    lower    upper 
#   1.7788   1.7702   1.7862

```

To obtain an estimated standard error, we can use the Wald method instead:
```r
invest(binom_fit, y0 = 0.5, interval = "Wald")

# estimate    lower    upper       se 
#   1.7788   1.7709   1.7866   0.0040

## Compare to 
MASS::dose.p(binom_fit, p = 0.5)
```

### Bioassays of nasturtium

```r

## Log-logistic model
log_fit <- nls(weight ~ theta1/(1 + exp(theta2 + theta3 * log(conc))),
               start = list(theta1 = 1000, theta2 = -1, theta3 = 1),
               data = nasturtium)
plotFit(log_fit, lwd.fit = 2)
           
## Inversion method
invest(log_fit, y0 = c(309, 296, 419), interval = "inversion")

# estimate    lower    upper 
#   2.2639   1.7722   2.9694

## Wald method
invest(log_fit, y0 = c(309, 296, 419), interval = "Wald")  

# estimate    lower    upper       se 
#   2.2639   1.6889   2.8388   0.2847
```

The intervals both rely on large sample results and normality. In practice, it the bootstrap may be more reliabile:
```r
## Bootstrap calibration intervals. In general, nsim should be as large as 
## reasonably possible (say, nsim = 9999).
boo <- invest(log_fit, y0 = c(309, 296, 419), boot = TRUE, nsim = 999, 
              seed = 101)
boo  # print bootstrap summary

# estimate       se     bias 
#   2.2639   0.2909   0.0320 
#
#  Percentile bootstrap interval: ( 1.8006 , 2.9336 )

plot(boo)  # plot results

```