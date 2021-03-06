---
layout: post
title: "Unsupervised Clustering- Classifying Wine"
author: "Maj"
date: "7/5/2020"
output: 
  html_document: 
    keep_md: yes
---






```r
summary(cars)
```

```
##      speed           dist       
##  Min.   : 4.0   Min.   :  2.00  
##  1st Qu.:12.0   1st Qu.: 26.00  
##  Median :15.0   Median : 36.00  
##  Mean   :15.4   Mean   : 42.98  
##  3rd Qu.:19.0   3rd Qu.: 56.00  
##  Max.   :25.0   Max.   :120.00
```

## Including Plots

You can also embed plots, for example:

![](Kmeansclustering-with-Wine_files/figure-html/pressure-1.png)<!-- -->

Note that the `echo = FALSE` parameter was added to the code chunk to prevent printing of the R code that generated the plot.

## The Data

We're gonna use UCI publically available datasets for red and white wine.
Let's check out what we have here.

```r
df1 <- read.csv("winequality-red.csv", sep=';')
df2 <- read.csv("winequality-white.csv", sep=';')
head(df1)
head(df2)
```

We're dealing with unsupervised learning here, and so we'll need to clean up data accordingly as will do below.

```r
df1$label <- sapply(df1$pH,function(x){'red'})
df2$label <- sapply(df2$pH,function(x){'white'})
head(df1)
# alright so we created the 'label' column
# merge dataframes now
wine <- rbind(df1,df2)
str(wine)
```

## Visualization the Data

Let's now do some viz work.

```r
library(ggplot2)
pl <- 
ggplot(wine,aes(x=residual.sugar)) + geom_histogram(aes(fill=label),color='black',bins=50) +
scale_fill_manual(values = c('#ae4554','#faf7ea')) +
theme_bw()
print(pl)
```
As we can see from the plot, red wine tends to have lower residual sugar levels, but we have to keep in mind that there are much fewer red wine datapoints, so there is a case for skewed data.

On to the next plot.

```r
pl <- ggplot(wine,aes(x=citric.acid)) + geom_histogram(aes(fill=label),color='black',bins=50) + scale_fill_manual(values = c('#ae4554','#faf7ea')) + theme_bw()
pl
```

Not much to learn here from acidity levels.

```r
pl <- ggplot(wine,aes(x=alcohol)) + geom_histogram(aes(fill=label),color='black',bins=50) 
+ scale_fill_manual(values = c('#ae4554','#faf7ea')) + theme_bw()
```

Nor from alcohol levels. There's much more white datapoints, so don't be fooled into making any rash conclusions.

Ok let's try to find features to use for our clustering model. Let's explore acidity and residual sugar?

```r
pl <- ggplot(wine,aes(x=citric.acid,y=residual.sugar)) + geom_point(aes(color=label),alpha=0.2) +
scale_color_manual(values = c('#ae4554','#faf7ea')) +theme_dark()
```

The plot shows you clustering wouldn't work given the chaos in the middle section.

Next, try volatile acidity versus residual sugar.

```r
pl <- ggplot(wine,aes(x=volatile.acidity,y=residual.sugar)) + geom_point(aes(color=label),alpha=0.2) +
scale_color_manual(values = c('#ae4554','#faf7ea')) +theme_dark()
```

Same here, these two features alone won't make it.

Ok let's set up our unsupervized dataset and feed it into our model.

```r
clus.data <- wine[,1:12]
head(clus.data)
```

## The Cluster Model

We now set up the Model.

```r
wine.cluster <- kmeans(wine[1:12],2)
# let's see the cluster means 
print(wine.cluster$centers)
```

## Evaluation

Normally, you don't have the red/white classifier info for each datapoint in unsupervized scenarios. But since we're simulating the scenario and have the classified data, we can actually do some evaluation of our model. Let's see how it fares.

```r
table(wine$label,wine.cluster$cluster)
```

Ok this isn't the best model. But it does a much, much better job at clustering red wine together, correctly that is.

Why this case study is flawed? For starters, the K-Means CLuster method does not tell me how many clusters it should use, nor what the labels (red or white) should be. So in fact, it is our knowledge of the data being restricted to red and white wine that made us go directly into using 2 K-clusters. In reality, without that knowledge, it is our task to know the nuances of wine, be experts at extracting knowledge from the data and use it to our advantage. Maybe the noise in white wine was simply due to the white wine category in the data including Rosé WIne datapoints? In that case, what if we used K = 3 clusters?

If there is one thing to learn from all of this, it is that you should know your wine ;)
