---
layout: post
title: "Predicting Loan Repayment with SVMs"
author: "Maj"
date: "7/4/2020"
output: 
  html_document: 
    keep_md: yes
---
## Deciding Whether to Extend a Bank Loan to a CLient with SVMs

Today, we will use a dataset from LendingClub.com, with which we examine loan profiles and outcomes, in order to develop a model that best predicts the ability of a new potential customer to repay a loan. The prediction, if accurate, can ultimately decide whether resources should be extended. This is important since banks need to ensure their money is paid back.

## The Data

Let's go ahead and load the data.
```r
ldata <- read.csv('loan_data.csv')
str(ldata)
summary(ldata)
```

## Some Tidying Up

```r
ldata$credit.policy <- factor(ldata$credit.policy)
ldata$inq.last.6mths <- factor(ldata$inq.last.6mths)
ldata$delinq.2yrs <- factor(ldata$delinq.2yrs)
ldata$pub.rec <- factor(ldata$pub.rec)
ldata$not.fully.paid <- factor(ldata$not.fully.paid)
str(ldata)
```
Much better!

## On to Some Visuals- Not Exactly Eye Candy but..;)

Intuition tells us that your credit score, FICO in the US, is a good indicator of your ability to repay and fulfill the loan repayment at maturity. But is it true here?

```r
library(ggplot2)
ficop <- ggplot(ldata,aes(x=fico)) 
ficop <- ficop + geom_histogram(aes(fill=not.fully.paid),color='black',bins=40,alpha=0.5)
ficop + scale_fill_manual(values = c('green','red')) + theme_bw()
```r

The data confirms this relationship. The higher the FICO score, the lower the proportion of loans that were not repaid.

Now, let's explore whether defaults are disproportionately represented within a specific category subset of loans- here we sort by the 'purpose' variable.

```r
purpl <- ggplot(ldata,aes(x=factor(purpose))) 
purpl <- purpl + geom_bar(aes(fill=not.fully.paid),position = "dodge")
purpl + theme_bw() + theme(axis.text.x = element_text(angle = 90, hjust = 1))
```

And finally, let's take a look at defaults in the broader context of the relationship between FICO scores and interest rates offered on the loans.

```r
ggplot(ldata,aes(int.rate,fico)) +geom_point(aes(color=not.fully.paid),alpha=0.3) + theme_bw()
```

Yes, more default-on-loan data points appear towards the bottom-right section of the graph. There, FICO scores are associated with higher interest rates charged. It's no surprise that this is how it played out.


## Setting Up an SVM

First, we get our training set, on which we set up the model.

```r
library(caTools)
set.seed(101)
spl = sample.split(ldata$not.fully.paid, 0.7)
train = subset(ldata, spl == TRUE)
test = subset(ldata, spl == FALSE)
library(e1071)
#
# SVM model
model <- svm(not.fully.paid ~ .,data=train)
summary(model)
```

## Predicting Loan Failures and Measuring Our Model's Accuracy

Let's get to know whether what we have is actually helpful. We predict outcomes for our test subset.

```r
predicted.values <- predict(model,test[1:13])
table(predicted.values, test$not.fully.paid)
```

Terrible! You really don't wanna base your loan extensions on this model. We need to fine-tune our parameters.


## Fine-Tuning the SVM Model

This process is dependent on some solid machine fire-power. I have used the formula below to test a simple range of parameters for the purposes of simplistic demonstration here only.

```r
tune.results <- tune(svm,train.x=not.fully.paid~., data=train,kernel='radial', ranges=list(cost=c(1,10), gamma=c(0.1,1)))
tune.results
```

As you can see, it is clear that the best parameters are cost = 10 and gamma = 0.1.
So we adopt those parameters in our model.

```r
model <- svm(not.fully.paid ~ .,data=train,cost=10,gamma = 0.1)
predicted.values <- predict(model,test[1:13])
table(predicted.values,test$not.fully.paid)
```

Much better predictions, our model is all set! 




Let's do this y'all
