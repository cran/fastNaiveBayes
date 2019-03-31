fastNaiveBayes
==============

[![CRAN status](https://www.r-pkg.org/badges/version/fastNaiveBayes)](https://cran.r-project.org/package=fastNaiveBayes) [![Travis build status](https://travis-ci.org/mskogholt/fastNaiveBayes.svg?branch=master)](https://travis-ci.org/mskogholt/fastNaiveBayes) [![Codecov test coverage](https://codecov.io/gh/mskogholt/fastNaiveBayes/branch/master/graph/badge.svg)](https://codecov.io/gh/mskogholt/fastNaiveBayes?branch=master) [![CRAN Downloads Total](http://cranlogs.r-pkg.org/badges/grand-total/fastNaiveBayes)](https://cran.r-project.org/package=fastNaiveBayes) [![CRAN Downloads Weekly](http://cranlogs.r-pkg.org/badges/last-week/fastNaiveBayes)](https://cran.r-project.org/package=fastNaiveBayes)

Overview
--------

This is an extremely fast implementation of a Naive Bayes classifier. This package is currently the only package that supports a Bernoulli distribution, a Multinomial distribution, and a Gaussian distribution, making it suitable for both binary features, frequency counts, and numerical features. Another unique feature is the support of a mix of different event models. Only numerical variables are allowed, however, categorical variables can be transformed into dummies and used with the Bernoulli distribution. This implementation offers a huge performance gain compared to the 'e1071' implementation in R. The execution times were compared on a data set of tweets and was found to be around 1135 times faster. Compared to other implementations the minimum speed up was found to be 12.5 times faster for the Bernoulli distribution. See the vignette for more details. This performance gain is only realized using a Bernoulli event model. Furthermore, the Multinomial event model implementation is even slightly faster, but incomparable since it was not implemented in 'e1071'. Compared to other implementations of a Multinomial distribution, this package was found to give a speed up of 12.2 times. The implementation is largely based on the paper "A comparison of event models for Naive Bayes anti-spam e-mail filtering" written by K.M. Schneider (2003).

Any issues can be submitted to: <https://github.com/mskogholt/fastNaiveBayes/issues>

Installation
------------

Install the package with:

``` r
install.packages("fastNaiveBayes")
```

Or install the development version using [devtools](https://github.com/hadley/devtools) with:

``` r
library(devtools)
devtools::install_github("mskogholt/fastNaiveBayes")
```

Usage
-----

``` r
rm(list=ls())
library(mlbench)
library(Matrix)
library(fastNaiveBayes)

# Load BreastCancer data
data(BreastCancer)
dim(BreastCancer)
levels(BreastCancer$Class)
head(BreastCancer)
# Select couple of columns
data_mat <- BreastCancer[,c("Class","Cl.thickness","Cell.size","Cell.shape","Marg.adhesion")]
y <- data_mat[,"Class"]
data_mat <- data_mat[,setdiff(colnames(data_mat),c("Class"))]
for(i in 1:ncol(data_mat)){
 data_mat[[i]] <- as.numeric(data_mat[[i]])
}
# Example using only Gaussian distribution
model <- fastNaiveBayes.mixed(data_mat[1:400,], y[1:400], laplace = 1, sparse = TRUE,
                             distribution = list(
                               gaussian = colnames(data_mat)
                             ))
preds <- predict(model, newdata = data_mat[401:nrow(data_mat),], type = "class")
mean(preds!=y[401:length(y)])
# Example mixing distributions
model <- fastNaiveBayes.mixed(data_mat[1:400,], y[1:400], laplace = 1, sparse = TRUE,
                             distribution = list(
                               multinomial = c("Cl.thickness","Cell.size"),
                               gaussian = c("Cell.shape","Marg.adhesion")
                             ))
preds <- predict(model, newdata = data_mat[401:nrow(data_mat),], type = "class")
mean(preds!=y[401:length(y)])
# Construct y and sparse matrix
# Bernoulli dummy example
data_mat <- BreastCancer[,c("Class","Cl.thickness","Cell.size","Cell.shape","Marg.adhesion")]
col_counter <- ncol(data_mat)+1
for(i in 2:ncol(data_mat)){
 for(val in unique(data_mat[,i])){
   data_mat[,col_counter] <- ifelse(data_mat[,i]==val,1,0)
   col_counter <- col_counter+1
 }
}
y <- data_mat[,"Class"]
data_mat <- data_mat[,setdiff(colnames(data_mat),c("Class","Cl.thickness", "Cell.size",
                                                  "Cell.shape","Marg.adhesion"))]
sparse_data <- Matrix(as.matrix(data_mat), sparse = TRUE)
data_mat <- as.matrix(data_mat)
# Example to estimate and predict once with Bernoulli distribution
model <- fastNaiveBayes.mixed(data_mat[1:400,], y[1:400], laplace = 1, sparse = TRUE,
                             distribution = list(
                               bernoulli = colnames(data_mat)
                             ))
preds <- predict(model, newdata = data_mat[401:nrow(data_mat),], type = "class")
mean(preds!=y[401:length(y)])
# Example using the direct model. This is much faster if all columns should have
# The same event model. It saves a lot of overhead
direct_model <- fastNaiveBayes.bernoulli(data_mat[1:400,], y[1:400], laplace = 1, sparse = TRUE)
direct_preds <- predict(direct_model, newdata = data_mat[401:nrow(data_mat),], type = "class")
mean(direct_preds!=y[401:length(y)])
```
