Overview
--------

This is a very fast implementation of the Naive Bayes classifier in R. It has the fastest execution time of any other Naive Bayes implementation in R. It's also the only implementation that makes it possible to use either a Bernoulli distribution or a multinomial distribution for the features.

Installation
------------

Development Version
-------------------

To get the development version, you can install this package directly from Github

``` r
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