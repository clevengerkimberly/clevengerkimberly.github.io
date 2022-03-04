---
layout: post
title: Calculating Activity Counts
subtitle: Using ActiGraph's Algorithm to Calculate Counts from Raw Acceleration Data
tags: [accelerometer, Python, raw acceleration]
---

I try (and fail) to implement ActiGraph’s activity count algorithm in Python.

---
## Our Purpose
Yesterday, ActiGraph released their previously-proprietary algorithm for calculating activity counts. This opens a world of possibility for physical activity researchers. But first, I just wanted to test out the algorithm.

## Implementing the Algorithm
ActiGraph released a [pre-print](https://www.researchsquare.com/article/rs-1370418/v1) describing the algorithm(s) and [Python code](https://github.com/actigraph/agcounts/) to calculate activity counts from raw acceleration data. I implemented the provided Python code in a [Jupyter notebook](https://jupyter.org/try) and in [Anaconda](https://www.anaconda.com/products/individual). I tested this with a few files- free-living, lab-based, using idle sleep mode versus no idle sleep mode, etc.  

Below I’m specifically showing two files that were including in the [GitHub repository](https://github.com/actigraph/agcounts/data.zip) ‘data.zip’ file:
- 'raw_30_30.csv’ in the ‘raw’ folder
- ‘raw_30_30_counts30sec.csv’ in the ‘ActiLifeCounts’ folder

I surmised (but could be totally wrong) that these two files were related. I assumed that if I used the count algorithm on the ‘raw_30_30.csv,’ I would get the same count values as found in the ‘raw_30_30_counts30sec.csv.’ The counts didn’t match, but I’m sure someone will either point out what I’ve done wrong, or re-implement the code based on the provided pseudo-code (hopefully in R!). For now, I’ll just show what I did and hopefully find an answer soon. 

Jupyter notebooks are super easy to use (I recomend that if you've never used Python) but below is what I did in Anaconda (a way of downloading/managing Python with a bunch of popular packages instead of doing it piece-meal). 

Open Anaconda from the Windows search bar and create an environment in which to work. 

```py 

conda create –name MyEnvironment 

conda activate MyEnvironment 

``` 


In this environment, I installed the required packages. 

```py 

conda install numpy 

conda install scipy 

conda install pandas 

``` 


Before we can run the ‘extract.py’ code file available on GitHub, I made some minor modifications. I added a few lines at the beginning to read in the raw acceleration file (raw_30_30.csv). 

```py 

from pandas import read_csv 

df = read_csv('raw_30_30.csv') 

data = df.values 

``` 


And added a few lines at the end to export the generated counts as a csv file. 

```py 

d=_extract(data,freq=30,epoch=30) 

np.savetxt("outputdata.csv",d,delimeter=",") 

``` 


Ok, now we can execute! 

```py 

python extract.py 

``` 


This writes a csv with count values- yay! 

[[4212 3645 4759] 

 [3766 4207 3734] 

 [4036 4064 3708] 

 ... 

 [4533 3891 4384] 

 [4111 4064 3809] 

 [3865 4183 4143]] 

  

Unfortunately, these count values do not match. The left three columns are from the ‘raw_30_30_counts30sec.csv’ file while the right three columns are from Python. Of note, they are in a different order (y, x, z vs. x, y, z). 

```r 
  Axis1. Axis2. Axis3. Col1. Col2. Col3.
1   3674   4184   4568  4212  3645  4759
2   4200   3773   3737  3766  4207  3734
3   4074   4036   3726  4036  4064  3708
4   4019   4302   3521  4304  4027  3523
5   3742   4118   4345  4123  3749  4330
6   3979   4015   3846  4023  3987  3844
  
```
  

When I have more time, I’ll keep looking in to this…otherwise I look forward to seeing how this develops. 


# That's it!
