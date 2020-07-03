---
layout: post
title: "Using Decision Trees and Random Forests to Identify Private/Public Uni's"
author: "Maj"
date: "7/3/2020"
output: 
  html_document: 
    keep_md: yes
---

## Intro

Today, we will be creating a model that should be able to classify schools as either public or private, based on data about the schools' features.

## The Dataset

We will be using the ISLR library, which possesses the College dataset.

```r
library(ISLR)
head(College,4)
df <- College
```

## What Intuition Tells Us, Plus A Road-Map

We already know that colleges in the US can vary tremendously across public/private lines. Private colleges are attended by a wealthier, whiter demographic, and tend to be smaller in size. Public uni's, on the other hand, portray a different set of characteristics. We will explore the data and confirm our initial thought process, as well as develop models that, with reasonable margin of error, correctly predict a college's public/private status based on its unique features.

## Exploring the Data

Let's first look into room board and graduation rates, and whether this relation changes along public/private lines.
```r
library(ggplot2)
plot1 <- ggplot(df,aes(Room.Board,Grad.Rate)) +
geom_point(aes(color=Private))
plot1
```
Next, let's check out whether size, measured by number of full time students, distinguishes public unis from private ones.

```r
plot2 <- ggplot(df,aes(F.Undergrad)) + geom_histogram(aes(fill=Private),color='black',bins=40)
print(plot2)
```

Clearly, private unis tend to be smaller while public ones are associated with bigger size (excluding part-time students)

What about graduation rates?

```r
plot3 <- ggplot(df,aes(Grad.Rate)) + geom_histogram(aes(fill=Private),color='black',bins=40)
plot3
```
As we can see, private unis have higher graduation rates, and this is more evident when we look at universities with the highest graduation rates.

There seems to be a university with a graduation rate above 100%, how can it be? Let's fix that.

```r
which.max(df$Grad.Rate)
df[96,]
```
Cazenovia College has a graduation rate of 118%, so smart ehh!
```r
df[96,]$Grad.Rate <- 100
df[96,]
```
Alright now! We move to modeling.

## Creating a Decision Tree Model

Let's now work with a subset of the data, on which we establish the model.
```r
library(caTools)
set.seed(101) 
cutoff = sample.split(df$Private, SplitRatio = .70)
train = subset(df, cutoff == TRUE)
test = subset(df, cutoff == FALSE)
```
Following, we set up our decision tree model to predict private status.
```r
library(rpart)
dtree <- rpart(Private ~.,method='class',data = train)
dtree.prediction <- predict(dtree,test)
head(dtree.prediction)

# change output of prediction table so that we join yes and no columns
dtree.prediction <- as.data.frame(dtree.prediction)
#
createclassifcolumn <- function(x){
    if (x>=0.5){
        return('Yes')
    }else{
        return("No")
    }
}
#
dtree.prediction$Private <- sapply(dtree.prediction$Yes,createclassifcolumn)
head(dtree.prediction)
```

And now, let's present the confusion matrix for our predicted outcomes, as well as plot the decision tree.
```r
table(dtree.prediction$Private,test$Private)
# plot tree
library(rpart.plot)
prp(dtree)
```

## Random Forest Model

Finally, we move on to this final section, in which we intend to set up a model using random forests and test our predictive abilities.

```r
library(randomForest)
rf.model <- randomForest(Private ~ . , data = train,importance = TRUE)
# confusion matrix
rf.model$confusion
```
And this is the feature importance of the model.
```r
rf.model$importance
```
The fun part now. Let's predict stuff!
```r
pred.rf <- predict(rf.model,test)
table(pred.rf,test$Private)
```
Not bad! Now, it is up to you to decide what type of error you wanna avoid/choose, and based on that you can judge the predictive abilities of the model.




Tis what it is,
Ciao y'all!


