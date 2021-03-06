[<img src="https://github.com/QuantLet/Styleguide-and-FAQ/blob/master/pictures/banner.png" width="888" alt="Visit QuantNet">](http://quantlet.de/)

## [<img src="https://github.com/QuantLet/Styleguide-and-FAQ/blob/master/pictures/qloqo.png" alt="Visit QuantNet">](http://quantlet.de/) **LM_JumpTest_2008** [<img src="https://github.com/QuantLet/Styleguide-and-FAQ/blob/master/pictures/QN2.png" width="60" alt="Visit QuantNet 2.0">](http://quantlet.de/)

```yaml

Name of QuantLet : LM_JumpTest_2008

Published in : 'To be published in METIS'

Description : 'R port of Python code to test whether a jump in a high frequency time series has happened by the methodology of Lee & Mykland (2008). In general, usage of this test is discouraged for high frequency datasets, e.g. tick data, as it is not rebust to microstructure noise. See Lee / Mykland (2012) and Ait-Sahalia / Jacod (2012) for methodologies that are robust to microstructure noise.'

Keywords : Jumps, jump test, high frequency, time series, Lee, Mykland, stochastic processes, cryptocurrencies, crypto

See also : ''

Author : Danial Florian Saef

Submitted : November 6 2019 by Danial Saef

```

### R Code
```r

# October 03, 2019
# By: Danial Saef
# R translation of Python code as found on
# https://gist.github.com/linuskohl/690da335a34ebf1cfc5ab27973e16ee5

if (!require(data.table)) install.packages("data.table")
library(data.table)

movmean <- function(v, kb, kf){
  # Computes the mean with a window of length kb+kf+1 that includes the element 
  # in the current position, kb elements backward, and kf elements forward.
  # Nonexisting elements at the edges get substituted with NaN.
  # 
  # Args:
  #     v (float) <- vector of values.
  #     kb (int) <- Number of elements to include before current position
  #     kf (int) <- Number of elements to include after current position
  #     
  # Returns:
  #     m <- vector of the same size as v containing the mean values
  
  
  m <- rep(NaN, length(v))
  
  for(i in kb:(length(v) - kf)) {
    m[i] <- mean(v[(i-kb):(i+kf+1)])
  }
  
  return(m)
}

LM_JumpTest_2008 <- function(S, sampling, significance_level=0.01){
  
  # "Jumps in Financial Markets: A New Nonparametric Test and Jump Dynamics"
  # - by Suzanne S. Lee and Per A. Mykland
  # 
  # "https://galton.uchicago.edu/~mykland/paperlinks/LeeMykland-2535.pdf"
  # 
  # Args:
  #     S <- An array containing prices, where each entry corresponds to the price sampled every "sampling" seconds.
  #     sampling (int) <- Seconds between entries in S
  #     significance_level (float) <- Defaults to 1% (0.001)
  #     
  # Returns:
  #     A data.table containing a row covering the interval 
  #     [t_i, t_i+sampling] containing the following values:
  #     J:   Binary value is jump with direction (sign)
  #     L:   L statistics
  #     T:   Test statistics
  #     sig: Volatility estimate
  
  tm <-  24*60*60 # Trading minutes
  k   <-  ceiling(sqrt(tm/sampling)) # block size
  
  r <-  c(NaN, diff(log(S)))
  
  bpv <-  abs(r) * abs(c(NaN, r[1:(length(r)-1)]))
  bpv <-  c(NaN, bpv[1:(length(r)-1)]) # Realized bipower variation
  sig <-  sqrt(movmean(bpv, k-3, 0)) # Volatility estimate
  L   <-  r/sig
  n   <-  length(S) # Length of S
  c   <-  (2/pi)**0.5
  Sn  <-  c*(2*log(n))**0.5
  Cn  <-  (2*log(n))**0.5/c - log(pi*log(n))/(2*c*(2*log(n))**0.5)
  beta_star   <-  -log(-log(1-significance_level)) # Jump threshold
  T   <-  (abs(L)-Cn)*Sn
  J   <-  as.numeric(T > beta_star)
  J   <-  J*sign(r) # Add direction
  # First k rows are NaN involved in bipower variation estimation are set to NaN.
  J[0:k] <-  NaN
  # Build and return result dataframe
  return (data.table('L' = L,'sig' = sig, 'T' = T,'J' = J))
  
}

DT_sample <- fread("DT_sample.csv")
LM_JumpTest_2008(DT_ts_p$p, 1)

```

automatically created on 2019-12-30