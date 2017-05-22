
<!-- README.md is generated from README.Rmd. Please edit that file -->
simsurv
=======

[![CRAN\_Status\_Badge](http://www.r-pkg.org/badges/version/simjm)](http://www.r-pkg.org/pkg/simjm) [![License](https://img.shields.io/badge/License-GPL%20%28%3E=%203%29-brightgreen.svg)](http://www.gnu.org/licenses/gpl-3.0.html)

**simjm** is an R package that allows the user to simulate data from a shared parameter joint model for longitudinal and time-to-event data.

The shared parameter joint model from which the simulated data is generated is based on the model formulation described for the `stan_jm` modelling function in the **rstanarm** R package. The shared parameter joint model can be univariate (one longitudinal marker) or multivariate (more than one longitudinal marker).

For the longitudinal submodel a generalised linear mixed model is assumed with any of the family choices allowed by glmer. If a multivariate joint model is specified, then the longitudinal submodel consists of a multivariate generalized linear model (GLM) with group-specific terms (at the individual-level) that are assumed to be correlated across the different GLM submodels. It is possible to specify a different outcome type (for example a different family and/or link function) for each of the GLM submodels, by providing a list of family objects in the family argument to the `simjm` call. Auxiliary parameters for each of the families can be specified via the `betaLong_aux` arugment to the `simjm` call. For each longitudinal marker submodel, the subject-specific longitudinal trajectory (on the linear predictor scale) can be specified as linear (`random_trajectory = "linear"`), quadratic (`random_trajectory = "poly"`) or none (`random_trajectory = "none"`); if the latter is used then a fixed-effect linear slope is still included but only an individual-level random intercept (i.e. no random slope).

Simulated event times are generated via a call to the **simsurv** package, which uses the approach described in Crowther and Lambert (2013) whereby the cumulative hazard is evaluated using numerical quadrature and survival times are generated using an iterative algorithm which nests the quadrature-based evaluation of the cumulative hazard inside Brent's (1973) univariate root finder. A variety of association structures can be specified for the joint model, however, the user is currently only allowed to choose one type of association structure for a given simulated dataset (i.e. only one association structure can be specified, but there are a variety of choices). Currently the only baseline hazard available for the event submodel is a Weibull distribution, but the "true" shape parameter can be specified by the user via the `betaEvent_aux` argument to the `simjm` call.

**Note:** This package was developed as an extension of the `simjm` function in the **survtd** package written by Margarita Moreno-Betancur, and described [here](https://github.com/moreno-betancur/survtd). The **survtd** package handles time-varying covariates in time-to-event models via a two-stage multiple imputation joint modelling approach. In comparison, the **simjm** package described here focuses on a variety of association structures for linking the longitudinal and event submodels, and a variety of family distributions for the longitudinal outcomes.

**Note:** Please note that the version available on GitHub is the most up-to-date *development* version of the package. A stable version of the package will be available from CRAN once it is released.

Getting Started
---------------

### Prerequisites

The **simjm** package includes the **simsurv** package as a dependency. The **simsurv** package can be downloaded from [here](https://github.com/sambrilleman/simsurv).

### Installation

You can install **simjm** directly from GitHub using the **devtools** package. To do this you should first check you have devtools installed by executing the following commands from within your R session:

``` r
if (!require(devtools)) {
  install.packages("devtools")
}
```

Then execute the following commands to install **simjm**:

``` r
library(devtools)
install_github("sambrilleman/simjm")
```

Example
-------

Here we show some basic calls to `simjm` to simulate various types of datasets.

Note that by default `simjm` will simulate data for three longitudinal biomarkers. However, if we only want one or two longitudinal biomarkers then we simply need to adjust the "true" parameters accordingly so that we still get realistic survival times. Some examples of appropriate "true" parameter values are given below.

``` r
# For one longitudinal marker:
simdat1 <- simjm(M = 1, betaEvent_assoc = 0.03)

# For two longitudinal markers:
simdat2 <- simjm(M = 2, betaEvent_assoc = 0.015)

# For three longitudinal markers, we can just use the defaults:
simdat3 <- simjm()

# Simulate three longitudinal markers, using "etaslope"
# association structure
simdat4 <- simjm(assoc = "etaslope", betaEvent_assoc = 0.3)
```

And here is an example of the resulting data frame:

``` r
simdat2[1:15,]
#>    id Z1       Z2 eventtime status        tij    Yij_1    Yij_2
#> 1   1  1 45.28698  8.052234      1 0.91105532 101.9608 122.5036
#> 2   1  1 45.28698  8.052234      1 3.61357931 107.0960 140.8192
#> 3   1  1 45.28698  8.052234      1 3.86710624 134.5808 140.7557
#> 4   1  1 45.28698  8.052234      1 4.89314573 135.1913 137.5043
#> 5   2  1 52.58583  7.258480      1 0.05075851 114.1853 144.0126
#> 6   2  1 52.58583  7.258480      1 2.17559970 127.8102 155.4861
#> 7   2  1 52.58583  7.258480      1 3.68861886 144.0486 143.8129
#> 8   2  1 52.58583  7.258480      1 4.40193697 116.1053 147.7632
#> 9   2  1 52.58583  7.258480      1 5.26379356 133.4231 154.8174
#> 10  3  1 44.88024  4.878875      1 3.16943950 174.8542 168.7051
#> 11  3  1 44.88024  4.878875      1 3.29525868 168.1846 146.9048
#> 12  3  1 44.88024  4.878875      1 3.77938550 171.4635 165.4653
#> 13  4  1 47.84656  3.214860      1 0.40972313 139.0482 165.4628
#> 14  4  1 47.84656  3.214860      1 1.02255119 132.5502 152.9094
#> 15  4  1 47.84656  3.214860      1 1.77074990 151.4380 174.4483
```

Bug Reports
-----------

If you find any bugs, please report them via email to [Sam Brilleman](mailto:sam.brilleman@monash.edu).

References
----------

1.  Crowther MJ, and Lambert PC. Simulating biologically plausible complex survival data. *Statistics in Medicine* 2013; **32**, 4118–4134. <doi:10.1002/sim.5823>

2.  Bender R, Augustin T, and Blettner M. Generating survival times to simulate Cox proportional hazards models. *Statistics in Medicine* 2005; **24(11)**, 1713-1723.

3.  Brent R. (1973) *Algorithms for Minimization without Derivatives*. Englewood Cliffs, NJ: Prentice-Hall.
