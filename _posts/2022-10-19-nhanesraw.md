---
layout: post
title: Downloading NHANES Raw Acceleration Data
subtitle: Using the ‘NHANES.RAW80Hz’ package
tags: [accelerometry, R, ActiGraph, surveillance]
---
Here I illustrate use of the ‘NHANES.RAW80Hz’ R package to download NHANES raw acceleration data.

## Our Purpose
In the United States, the National Health and Nutrition Examination Survey (NHANES) is one of our primary surveillance systems for capturing the health and behavior of children, adolescents, and adults. In the 2011-2012 and 2013-2014 cycles, participants were asked to wear a wrist-worn ActiGraph GT3X+ on their non-dominant wrist. These data were also available for the National Youth Fitness Survey (NYFS), a one-time extension to NHANES which focused on children and adolescents. These accelerometer data were initially released as Monitor-Independent Movement Summary (MIMS) units at the minute, hour, and day level (e.g., see [the post](https://clevengerkimberly.github.io/2022-04-09-nhanes/) I have on that data) . However, the raw acceleration data at a sampling frequency of 80 Hz were later made available. Because of the size of the data (almost 30 Tb in total), they opted to release these data as individual files per hour for each participant. While you can click and download each file, that would take forever…..
The easier way is using the R package ‘NHANES.RAW80Hz’ which was developed and released by the National Cancer Institute (part of the National Institutes of Health). Below, I’ll show you how this package can be used to download all accelerometer files for a list of participants. I won't show you all the steps that come next, like actually analyzing the data, using survey weights, etc. If you have questions about that, feel free to reach out!

## Download the package
The package is available as a [zip file.](https://epi.grants.cancer.gov/physical/NHANES-RAW80Hz-0.0.3.zip) The vignette is also available [online.](https://epi.grants.cancer.gov/physical/vignette.html) 
You can then install the package in R (Packages -> install -> change ‘install from’ to ‘Package Archive file’)

## Identifying Participants
In this example, I only wanted data from NYFS participants who were 6-11 y of age and had accelerometer data. So, I downloaded 1) the [demographics file](https://wwwn.cdc.gov/Nchs/Nnyfs/Y_demo.htm), which contains information like age, and 2) the [‘PAM Header File',](https://wwwn.cdc.gov/Nchs/Nnyfs/Y_PAXHD.htm#PAXSTS) which contains a variable ‘PAXSTS’ that is ‘1’ when the person has accelerometer data and '0' when they do not.

I first read these files in to R, using the ‘haven’ package (because they are in ‘xpt’ format). In the same step, I use the ‘dplyr’ package to select only the variables I want to keep (it can get unwieldy otherwise!). These are the participant’s id (SEQN), age (RIDAGEYR), and whether they have accelerometer data (PAXSTS). If I wanted, I could use other exclusion criteria at this step, like health status, whether they responded to a specific question of interest, etc. Because the data are so big, I highly encourage you to only download the accelerometer data you will actually use!

```r
tryCatch(library(haven),error=function(e){install.packages("haven");library(haven)})
tryCatch(library(dplyr),error=function(e){install.packages("dplyr");library(dplyr)})
demo<-haven::read_xpt("Y_DEMO.xpt") %>% select("SEQN","RIDAGEYR")
pahead<-haven::read_xpt("Y_PAXHD.xpt") %>% select("SEQN","PAXSTS")
```

I then merge the two files together and create a list of id numbers (“SEQN”) which correspond to children 6-11 y of age with accelerometer data. I end up with 701 participants in this example, which I make a vector called ‘seqn’ for use in the next step.
```r
data<-merge(demo,pahead,by="SEQN",all=TRUE) %>% subset(PAXSTS==1 & RIDAGEYR<12 & RIDAGEYR>5)
seqn<-as.vector(data$SEQN)
```

I can then use the ‘NHANES.RAW80Hz’ package to download all the files corresponding to these id numbers (found in ‘seqn’). Here, a few things need to be defined, such as the place you want the files to be saved ('desDir'), which dataset you are interested in (here I am using 'NNYFS_2012'). I use the 'subject_seqn' argument to tell the function I only want files from my list of id numbers, ‘seqn.’ Lastly, if you are on a Windows computer, you have to use 7-zip or WinZip to extract the downloaded files and you have to tell the function where to find this program on your computer.
```r
library(NHANES.RAW80Hz)
setwd("C:/Users/clevengerka/nyfs/")
destDir<-getwd()
zipcmd<-“C:/Program Files/7-Zip/7z.exe”
downloadAcclRawData80Hz(destDir, data="NNYFS_2012", subject.seqn=c(seqn), zipcmd=zipcmd)
```

Now what I end up with is a bunch of individual folders on my computer, one for each id number in ‘seqn.’ Within each of these folders, there is one file per hour of raw acceleration data. I can get a list of these files using the code below. 

```r
files <- as.data.frame(list.files(path = " C:/Users/clevengerka/nyfs/NNYFS_2012/", pattern = "^GT3XPLUS-AccelerationCalibrated", recursive = TRUE, full.names = TRUE)) 
```
This is helpful because I can then use the ‘lappy’ function to apply a function over all the files, and bind them in to one giant data set. In this case, I made a function which reads the file, calculates ActiGraph counts and ENMO over a 5-sec epoch, then saves the file with a shorter filename, but you can do whatever you want!
Good luck!
