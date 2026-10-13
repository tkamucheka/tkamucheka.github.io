---
layout: project
title: CuSMC
project: cusmc
---

CuSMC is an R package for drawing samples from a posterior probability
distribution in Bayesian inference. Its name stands for CUDA Sequential Monte
Carlo. It provides a Metropolis-Hastings sampler and multivariate normal and
Student-t distributions.

A GPU is not required: without an NVIDIA GPU and the CUDA toolkit, CuSMC is
built for the CPU only. On Linux with an NVIDIA GPU and the CUDA toolkit, it
is built with GPU acceleration. macOS supports CUDA only up to macOS
10.13.6, and Windows builds are CPU-only.

## Installation

CuSMC is built from source, which requires a C++ toolchain (on Windows,
[Rtools](https://cran.r-project.org/bin/windows/Rtools/)) and the R packages
`devtools`, `Rcpp`, and `RcppEigen`:

```r
install.packages(c("devtools", "Rcpp", "RcppEigen"))
devtools::install_github("tkamucheka/cusmc")
library(CuSMC)
```

Platform-specific requirements are described in the
[README](https://github.com/tkamucheka/CuSMC#readme).

## Publication

* **Kamucheka, T.**, Mai, Q., Wei, G., Huang, M., & Liu, X.
  CuSMC: Fast Parallel Implementation for Sequential Monte Carlo on GPU, R
  package.
