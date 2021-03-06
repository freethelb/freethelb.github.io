---
title: 'Identifying Fake Banknotes with Neural Networks'
author: "Maj"
date: "7/15/2020"
output: 
  html_document: 
    keep_md: yes
---



## R Markdown

This is an R Markdown document. Markdown is a simple formatting syntax for authoring HTML, PDF, and MS Word documents. For more details on using R Markdown see <http://rmarkdown.rstudio.com>.

When you click the **Knit** button a document will be generated that includes both content as well as the output of any embedded R code chunks within the document. You can embed an R code chunk like this:


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

![](Identifying-Fake-Banknotes-with-Neural-Networks_files/figure-html/pressure-1.png)<!-- -->

Note that the `echo = FALSE` parameter was added to the code chunk to prevent printing of the R code that generated the plot.

## The Data
```r
df <- read.csv('bank_note_data.csv')
df
```

## Let's Design Our ANN Model

```r
# split the data into train and test sub-sets
library(caTools)
split <- sample.split(df$Class, SplitRatio = 0.7)
train <- subset(df, split == TRUE)
test <- subset(df, split == FALSE)
```
Now we have a subset of the data with which to train our neural network.

Let's go on and train an ANN model with our "train" data.
```r
library(neuralnet)
nn <- neuralnet(Class ~ Image.Var + Image.Skew + Image.Curt + Entropy, data = train, hidden = c(5,3), linear.output = FALSE)
```

Aha! On to the fun part. Prediction it is!

```r
predicted.nn.values <- compute(nn,test[,1:4])
head(predicted.nn.values$net.result)
predictions <- sapply(predicted.nn.values$net.result, round)
head(predictions)
```
## Assessing Our Model's Predictions

```r
table(predictions,test$Class)
```
The confusion matrix above indicates a 100% correct prediction rate. Is there such a thing as a perfect model? We social scientists won't buy it, at least not so fast.

Let's compare the model to a Random Forest model.
We perform splitting of the dataset once again, while treating Class as a factor to accommodate for the Random Forest model.
```r
library(randomForest)
df$Class <- factor(df$Class)
library(caTools)
set.seed(101)
split = sample.split(df$Class, SplitRatio = 0.70)

train = subset(df, split == TRUE)
test = subset(df, split == FALSE)
```

Ok let's train the model now.
```r
model <- randomForest(Class ~ Image.Var + Image.Skew + Image.Curt + Entropy,data=train)
```
On to prediction with the now-ready random forest model.
```r
rf.pred <- predict(model,test)
```
And then, we're ready to verify the predictive powers of our random forest model.
```r
table(rf.pred,test$Class)
```
Taraaa! The random forest model predicts almost 100% correctly, which is evidence that our initial Artificial Neural Net model is actually truly powerful. Not a coincidence really.


May the Power of Data be with You!


