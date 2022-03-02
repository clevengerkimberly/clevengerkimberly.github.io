---
layout: post
title: Repository - Steenbock
subtitle: Applying the Steenbock Neural Network to Raw Acceleration Data
tags: [accelerometry, R, raw acceleration, repository]
---
In this series of posts, I’ll show how to use models available in our repository of methods for analyzing accelerometer data.
---

## Our Purpose
If you haven’t already, check out our [repository](https://sites.google.com/view/accelerometerrepository/) which hosts 57 methods (soon to be more!) for analyzing accelerometer data using raw acceleration or novel analytic techniques like machine learning. The purpose of the repository is to make models easier for researchers to find and implement, ideally so these models can be further cross-validated or developed. 
Each model in the repository has information like how it was developed, instructions for use, and, when possible, example data and R code.
Today I’ll illustrate how to use one of the models published by [Steenbock et al. (2019)](https://sites.google.com/view/accelerometerrepository/available-models/provided-as-r-code/steenbock). 
Steenbock et al. (2019) did a great job of providing example data for their models. However, today I want to start from raw acceleration data, calculate features, and then implement the model.


## Calculating Features
First we load some packages. As an aside, you have to install and load packages in R before you can use them. You can use library(mlr), for example, to load the mlr package if you already have it installed. The code below tries to load the mlr package, for example, but if it fails (because it isn’t installed), it will install and then load the package. I prefer this because then I know that the packages will be there no matter who/what computer is running the code.
```r
tryCatch(library(mlr),error=function(e){install.packages("mlr");library(mlr)})
tryCatch(library(data.table),error=function(e){install.packages("data.table");library(data.table)})
tryCatch(library(lme4),error=function(e){install.packages("lme4");library(lme4)})
tryCatch(library(ranger),error=function(e){install.packages("ranger");library(ranger)}) 
```

Steenbock et al. (2019) actually provided like…72 models in this paper, but let’s just focus on one for now- the ActiGraph right hip neural network. We first load the model and see which features it wants. This model object (filename 'nn_actigraph_right_METs_30s.Rds') was provided by Steenbock et al., (2019); further description is in their original paper and on the repository site.
```r
nn_actigraph_right_model<-readRDS("nn_actigraph_right_METs_30s.Rds")
nn_actigraph_right_model$features
[1] "X.mean" "X.sd"   "X.min"  "X.max"  "X.cov"  "X.p10"  "X.p25"  "X.p50"  "X.p75"  "X.p90"  "Y.mean"
[12] "Y.sd"   "Y.min"  "Y.max"  "Y.cov"  "Y.p10"  "Y.p25"  "Y.p50"  "Y.p75"  "Y.p90"  "Z.mean" "Z.sd"  
[23] "Z.min"  "Z.max"  "Z.cov"  "Z.p10"  "Z.p25"  "Z.p50"  "Z.p75"  "Z.p90"
```

On the repository website, you can see the example data sheet provided by Steenbock et al. (2019) but this assumes you already have the features calculated. 
Instead, let’s read in a gt3x raw acceleration file and calculate these features ourselves in 30-s non-overlapping windows.
To do this, we are going to define a function called 'feature_extraction' and then apply it to our data.
```r
rawdata<-as.data.frame(read.gt3x::read.gt3x("File1.gt3x"))
rawdata$Timestamp<-lubridate::floor_date(rawdata$time,"30 sec")
setDT(rawdata)

feature_extraction <- function(x){
  percentiles <- as.list(quantile(x, c(0.1, 0.25, 0.5, 0.75, 0.9), names = FALSE))
  names(percentiles) <- c("p10", "p25", "p50", "p75", "p90")
  cov <- acf(x, lag.max = 1, plot = FALSE)$acf[2, , ]
  if (is.na(cov)) {
    cov <- 0
  }
  names(cov)<-c("cov")
  list(mean = mean(x), sd = sd(x), min=min(x), max=max(x),cov,percentiles)
}

data30sec<-rawdata[, as.list(unlist(lapply(.SD, feature_extraction))), by="Timestamp", .SDcols=c("X","Y","Z")]
```


## Predicting METs
Now that we have our 30-s feature data, we can use the 'nn_actigraph_right_model' model object provided by Steenbock et al. (2019) to predict METs for each epoch, which we will bind back to the 30-s data.
```r
data30sec<-as.data.frame(data30sec)
pred <-as.vector(predict(nn_actigraph_right_model, newdata = data30sec)$data$response)
data30sec<-cbind(data30sec,pred)
```

## Putting it All Together
But, let’s be realistic. You don’t normally only have one file from one participant. Here’s an example of how I might apply the Steenbock algorithm to multiple participants. In addition to the ‘feature_extraction’ function defined above, I’m going to make a ‘steenbock’ function which will read in a gt3x file, calculate the features, and use the model to predict METs. You can probably imagine how this could be tweaked slightly to work for other types of raw acceleration data (e.g., ActiGraph raw csv, GENEActiv, etc.).
```r

steenbock<-function(x){
  rawdata<-as.data.frame(read.gt3x::read.gt3x(x))
  rawdata$id
  rawdata$Timestamp<-lubridate::floor_date(rawdata$time,"30 sec")
  setDT(rawdata)
  
  data30sec<-rawdata[, as.list(unlist(lapply(.SD, feature_extraction))), by="Timestamp", .SDcols=c("X","Y","Z")]
  
  data30sec<-as.data.frame(data30sec)
  pred <-as.vector(predict(nn_actigraph_right_model, newdata = data30sec)$data$response)
  data30sec<-cbind(data30sec,pred)
  return(data30sec)
}
```

I can then apply this function to a whole list of files in my working directory.
```r
filenames<-list.files(pattern="*.gt3x")
data<-try(lapply(filenames,steenbock) %>% bind_rows())
```


## The End
