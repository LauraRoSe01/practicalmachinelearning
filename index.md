---
title: "Practical Machine Learning Project"
author: "Laura Robledo"
date: "31/07/2020"
  html_document: default
  pdf_document: default
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```


```{r, warning=FALSE, error=FALSE, echo=FALSE, message=FALSE}
library(corrplot)
library(caret)
library(readr)
```


# Predicting "how well" an activity has been done by using data obtained from Human Activity Recognition systems


## Summary


The goal of this project is to create a model to predict if an exercise has been done properly based on the data collected from different sensors.  

For this, we will be using data on six young healthy male participants aged between 20-28 years that were asked to perform one set of 10 repetitions of the Unilateral Dumbbell Biceps Curl in five different fashions: 
**Class A** -- Exactly according to the specification  
**Class B** -- Throwing the elbows to the front  
**Class C** -- Lifting the dumbbell only halfway  
**Class D** -- Lowering the dumbbell only halfway  
**Class E** -- Throwing the hips to the front  



## Data

Training data:  

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv  

Test data:  

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv  

Ugulino, W.; Cardador, D.; Vega, K.; Velloso, E.; Milidiu, R.; Fuks, H. Wearable Computing: Accelerometers' Data Classification of Body Postures and Movements. Proceedings of 21st Brazilian Symposium on Artificial Intelligence. Advances in Artificial Intelligence - SBIA 2012. In: Lecture Notes in Computer Science. , pp. 52-61. Curitiba, PR: Springer Berlin / Heidelberg, 2012. ISBN 978-3-642-34458-9. DOI: 10.1007/978-3-642-34459-6_6.  

Read more: http://groupware.les.inf.puc-rio.br/har#ixzz6TfIvKmk7  
           http://groupware.les.inf.puc-rio.br/har#ixzz6TfJHbAHG  


## Data Manipulation

Only the columns with information regarding the metrics obtained by the sensors and the class of the activity have been kept:

```{r, cache=TRUE, echo=FALSE, warning=FALSE, error=FALSE, message=FALSE}
pml_training <- read_csv("pml-training.csv")
pml_testing <- read_csv("pml-testing.csv") 

```

```{r}
names(pml_testing)
```


## Training and testing sets/Cross-referencing 

70% of the data was used for the training data set, the rest was allocated to the testing data set.

```{r}
set.seed(1)
inTrain <- createDataPartition(pml_training$classe, p = 0.7, list = FALSE)
trainData <- pml_training[inTrain, ]
testData <- pml_training[-inTrain, ]
dim(trainData)
dim(testData)
```



## Exploratory plots

```{r, cache=TRUE, echo=FALSE}
pie <- as.data.frame(table(trainData$classe))
pie(pie$Freq, labels = pie$Var1, main = "Share of Activity Class in Training Set", col=rainbow(length(5:10)))
res <- cor(trainData[,1:51])
corrplot(res, type="lower", method = "circle", main= "Correlation Between Variables")
```

From the correlation plot we can see that there are only a few variables that have strong correlations.


## Prediction models


Three different models were compared:  
-- Random Forests ("rf")  
-- Boosted Trees ("gbm")  
-- Linear Discriminant analysis ("lda") 

Finally, the predictions are Stacked together using random forests ("rf"). 

```{r, cache=TRUE, results="hide"}
mod_rf <- train(classe ~ ., data = trainData, method = "rf") #Creating models
mod_gbm <- train(classe ~ ., data = trainData, method = "gbm")
mod_lda <- train(classe ~ ., data = trainData, method = "lda")
pred_rf <- predict(mod_rf, testData) #Testing models using test dataset
pred_gbm <- predict(mod_gbm, testData)
pred_lda <- predict(mod_lda, testData)

predDF <- data.frame(pred_rf, pred_gbm, pred_lda, classe = testData$classe) #combine models & predict
combModFit <- train(classe ~ ., method = "rf", data = predDF)
combPred <- predict(combModFit, predDF)

```


# Measure Accuracy in the Models

```{r}
#Random Forest
round(confusionMatrix(pred_rf, as.factor(testData$classe))$overall[1],2)
#Boosted Trees
round(confusionMatrix(pred_gbm, as.factor(testData$classe))$overall[1],2)
#Linear Discriminant analysis
round(confusionMatrix(pred_lda, as.factor(testData$classe))$overall[1],2)
#combined 
round(confusionMatrix(combPred, as.factor(testData$classe))$overall[1],2)
```

## The Random Forest model has been selected since it has just as high accuracy as the combined model equal to 0.99.  

```{r, echo=FALSE}
cm <- confusionMatrix(pred_rf, as.factor(testData$classe))
plot(cm$table, col = cm$byClass, main = "Random Forest", round(cm$overall['Accuracy'], 2))
```


## Calculating Out of Sample Error


```{r}
dim(testData)
```

```{r}
prediction <- predict(mod_rf, newdata=testData)
length(prediction)
```

```{r}
round(100*(1-sum(prediction == testData$classe)/length(prediction)),2)
```

The out of sample error is expected to be around 0.8%


## Prediction 20 cases (Quiz)

Finally, the original test data provided for testing is used to create the predictions for the quiz.

```{r}
predict(mod_gbm, newdata=pml_testing)
```

