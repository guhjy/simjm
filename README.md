
<!-- README.md is generated from README.Rmd. Please edit that file -->
simjm
=====

[![CRAN\_Status\_Badge](http://www.r-pkg.org/badges/version/simjm)](http://www.r-pkg.org/pkg/simjm) [![License](https://img.shields.io/badge/License-GPL%20%28%3E=%203%29-brightgreen.svg)](http://www.gnu.org/licenses/gpl-3.0.html)

**simjm** is an R package that allows the user to simulate data from a shared parameter joint model for longitudinal and time-to-event data.

The shared parameter joint model from which the simulated data is generated is based on the model formulation described for the `stan_jm` modelling function in the **rstanarm** R package. See [this](https://cran.r-project.org/web/packages/rstanarm/vignettes/jm.html) vignette for some of the details. The shared parameter joint model can be univariate (one longitudinal marker) or multivariate (more than one longitudinal marker).

For the longitudinal submodel a generalised linear mixed model is assumed with any of the family choices allowed by `stats::glm`. If a multivariate joint model is specified, then the longitudinal submodel consists of a multivariate generalized linear model (GLM) with group-specific terms (at the individual-level) that are assumed to be correlated across the different GLM submodels. It is possible to specify a different outcome type (for example a different family and/or link function) for each of the GLM submodels, by providing a list of family objects in the family argument to the `simjm` call. Auxiliary parameters for each of the families can be specified via the `betaLong_aux` arugment to the `simjm` call. For each longitudinal marker submodel, the population-level longitudinal trajectory is determined via the `fixed_trajectory` argument. This can be specified as either `"none"` (intercept only), `"linear"` (intercept and linear slope), `"quadratic"` (intercept, linear slope, and quadratic term), or `"cubic"` (the default, intercept, linear slope, quadratic term, and cubic term). In addition, the subject-specific deviations from this trajectory (i.e. subject-specific parameters, also known as subject-level random effects) are determined via the `random_trajectory` argument (specified in a similar manner to the `fixed_trajectory` argument).

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

And here is an example of the resulting data frames:

``` r
simdat2$Event %>% dplyr::filter(id %in% 1:2)
#>   id Z1         Z2 eventtime status
#> 1  1  0 -1.6072995  20.00000      0
#> 2  2  0 -0.7013221  12.51011      1
simdat2$Long1 %>% dplyr::filter(id %in% 1:2)
#>    id Z1         Z2 eventtime status        tij    Yij_1
#> 1   1  0 -1.6072995  20.00000      0  0.0000000 7.407485
#> 2   1  0 -1.6072995  20.00000      0  7.8174792 8.248891
#> 3   1  0 -1.6072995  20.00000      0  9.0557943 7.174677
#> 4   1  0 -1.6072995  20.00000      0  9.3296569 7.241487
#> 5   1  0 -1.6072995  20.00000      0  9.4837721 8.056077
#> 6   1  0 -1.6072995  20.00000      0 14.5213306 7.084750
#> 7   1  0 -1.6072995  20.00000      0 17.8353023 5.609793
#> 8   1  0 -1.6072995  20.00000      0 18.2803002 6.196316
#> 9   1  0 -1.6072995  20.00000      0 19.3026583 5.134052
#> 10  1  0 -1.6072995  20.00000      0 19.7686089 5.413853
#> 11  2  0 -0.7013221  12.51011      1  0.0000000 7.930274
#> 12  2  0 -0.7013221  12.51011      1  0.9755494 7.564502
#> 13  2  0 -0.7013221  12.51011      1  2.7564974 8.828539
#> 14  2  0 -0.7013221  12.51011      1  2.9984159 7.187226
#> 15  2  0 -0.7013221  12.51011      1  5.2723265 7.329603
#> 16  2  0 -0.7013221  12.51011      1  7.1082889 7.550815
#> 17  2  0 -0.7013221  12.51011      1 10.9639076 7.311300
simdat2$Long2 %>% dplyr::filter(id %in% 1:2)
#>    id Z1         Z2 eventtime status         tij     Yij_2
#> 1   1  0 -1.6072995  20.00000      0  0.00000000  8.946639
#> 2   1  0 -1.6072995  20.00000      0  0.06402065  8.782643
#> 3   1  0 -1.6072995  20.00000      0  0.06828614 10.131194
#> 4   1  0 -1.6072995  20.00000      0  1.00358446  8.953131
#> 5   1  0 -1.6072995  20.00000      0  2.85765766  9.108531
#> 6   1  0 -1.6072995  20.00000      0  7.85927276  7.711013
#> 7   1  0 -1.6072995  20.00000      0 11.98695763  7.932241
#> 8   1  0 -1.6072995  20.00000      0 14.67148988  8.327863
#> 9   1  0 -1.6072995  20.00000      0 19.34600598  6.102380
#> 10  1  0 -1.6072995  20.00000      0 19.60524346  5.205007
#> 11  2  0 -0.7013221  12.51011      1  0.00000000 10.182755
#> 12  2  0 -0.7013221  12.51011      1  2.88195474  9.638174
#> 13  2  0 -0.7013221  12.51011      1  4.60165111  9.835940
#> 14  2  0 -0.7013221  12.51011      1 12.02851147  8.827208
```

Bug Reports
-----------

If you find any bugs, please [file an issue](https://github.com/sambrilleman/simjm/issues) or report them via email to [Sam Brilleman](mailto:sam.brilleman@monash.edu).

References
----------

1.  Crowther MJ, and Lambert PC. Simulating biologically plausible complex survival data. *Statistics in Medicine* 2013; **32**, 4118--4134. <doi:10.1002/sim.5823>

2.  Bender R, Augustin T, and Blettner M. Generating survival times to simulate Cox proportional hazards models. *Statistics in Medicine* 2005; **24(11)**, 1713--1723.

3.  Brent R. (1973) *Algorithms for Minimization without Derivatives*. Englewood Cliffs, NJ: Prentice-Hall.
