---
layout: default
---


```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## A Random Forest example using the Iris dataset in R

In this document I will show a simple example of using Random Forest to make some predictions.
First I will do some data exploration using the IRIS dataset, including Principal Component Analysis using `prcomp`.
The Iris dataset is included in R core.
The Random Forest example is at the end of the document.


Most of the RF code found here (http://rischanlab.github.io/RandomForest.html)  
For PCA I took inspiration from https://www.r-bloggers.com/computing-and-visualizing-pca-in-r/



### Some preliminary exploratory analysis of the iris dataset
```{r ggplot2}
 set.seed(12345)
 data(iris)
 dim(iris)
 iris[1:5,1:5]
 table(iris$Species)
 boxplot(iris$Sepal.Length ~ iris$Species,ylab="Sepal.Length")
 boxplot(iris$Sepal.Width ~ iris$Species,ylab="Sepal.Width")
 boxplot(iris$Petal.Length ~ iris$Species,ylab="Petal.Length")
 boxplot(iris$Petal.Width ~ iris$Species,ylab="Petal.Width")
 
```


We can see that petal descriptors show more differentiation betwen species tat sepal descriptors and that virginica and versicolor are closer to each other than to setosa for most variables.

### A simple plot using petal descriptors

```{r}
 plot(iris$Petal.Length,iris$Petal.Width,col=iris$Species,pch=16)
 legend( x="topleft", 
        legend=levels(as.factor(iris$Species)),
        col=c("black","red","green"), 
        pch=c(16) )
 
```

### Using PCA to show differentiation between species. We won't try to predict anything with PCA for this example.

```{r}
log.ir <- log(iris[, 1:4])
ir.species <- iris[, 5]
 
ir.pca <- prcomp(log.ir,
                 center = TRUE,
                 scale. = FALSE) 
ir.pca
pca.results<-cbind(as.data.frame(ir.pca$x),ir.species)
head(pca.results)
plot(pca.results$PC1,pca.results$PC2,col=pca.results$ir.species,pch=16)
 legend( x="topleft", 
        legend=levels(as.factor(pca.results$ir.species)),
        col=c("black","red","green"), 
        pch=c(16) )
```

We notice the species separate over PC1. Still some overlp between versicolor and virginica.

## Now using the Random Forest  method

###First we split iris data into a training and a testing set
70% of the data will be used for training the model.

```{r}
ind <- sample(2,nrow(iris),replace=TRUE,prob=c(0.7,0.3))
table(ind)
trainData <- iris[ind==1,]
testData <- iris[ind==2,]
dim(trainData)
dim(testData)
head(trainData)
head(testData)
```
### Loading randomForest package and generating the Random Forest Learning Tree
If the package has not been installed previously you might need to install it with `install.packages("randomForest")`
```{r, message=FALSE, warning=FALSE}
library(randomForest)
iris_rf <- randomForest(Species~.,data=trainData,ntree=100,proximity=TRUE)
print(iris_rf)
#table(predict(iris_rf),trainData$Species)
```

We can see that usin the current training data RF has no problem identifying the setosa individuals (class.error=0) but there is more uncertainty for assignment to the classes versicolor and virginica (clas.error ~ 0.08).

## Importance of the class descriptors
```{r}
importance(iris_rf)
```

We can see that Petal.width and Petal.length are the more important descriptors that differentiate between species as shown before by PCA and just plotting this variables.

## Now we build a random forest for our testing data
```{r}
  irisPred<-predict(iris_rf,newdata=testData)
  table(irisPred, testData$Species)
```
## Plotting the margins. Positive margins mean correct classification.

```{r}
plot(margin(iris_rf,testData$Species))
```

## And checking the classification accuracy
```{r}
  #The number of correct predictions
  print(sum(irisPred==testData$Species))
  #out of this many datapoints used to test the prediction
  print(length(testData$Species)) 
 
  #the accuracy
  print(sum(irisPred==testData$Species)/length(testData$Species))
```
