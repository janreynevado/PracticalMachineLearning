---
title: "Practical Machine Learning Course Project"
author: "Janrey Nevado"
date: "August 12, 2019"
output: html_document
---

```{r setup, include = FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## Background

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. The goal of this project will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants of the study<sup>1</sup>. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. 

More information and data are available from the source website of the study<sup>1</sup>: http://groupware.les.inf.puc-rio.br/har

<pre>[1] <i>Ugulino, W.; Cardador, D.; Vega, K.; Velloso, E.; Milidiu, R.; Fuks, H.</i> Wearable Computing: Accelerometers' Data Classification of Body Postures and Movements. Proceedings of 21st Brazilian Symposium on Artificial Intelligence. Advances in Artificial Intelligence - SBIA 2012. In: Lecture Notes in Computer Science. , pp. 52-61. Curitiba, PR: Springer Berlin / Heidelberg, 2012. ISBN 978-3-642-34458-9. DOI: 10.1007/978-3-642-34459-6_6.</pre>

***

## Data

* The training data for this project are available here: [https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv]
* The test data are available here: [https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv]
* The data for this project come from this source: [http://groupware.les.inf.puc-rio.br/har].

***

## Approach

Our outcome variable is classe, a factor variable. For this data set, “participants were asked to perform one set of 10 repetitions of the Unilateral Dumbbell Biceps Curl in 5 different fashions: - exactly according to the specification (Class A) - throwing the elbows to the front (Class B) - lifting the dumbbell only halfway (Class C) - lowering the dumbbell only halfway (Class D) - throwing the hips to the front (Class E)

Two models will be tested using decision tree and random forest. The model with the highest accuracy will be chosen as our final model.

***

### Cross-validation

Cross-validation will be performed by subsampling our training data set randomly without replacement into 2 subsamples: TrainTrainingSet data (75% of the original Training data set) and TestTrainingSet data (25%). Our models will be fitted on the TrainTrainingSet data set, and tested on the TestTrainingSet data. Once the most accurate model is chosen, it will be tested on the original Testing data set.

***

### Expected out-of-sample error

The expected out-of-sample error will correspond to the quantity: 1-accuracy in the cross-validation data. Accuracy is the proportion of correct classified observation over the total sample in the TestTrainingSet data set. Expected accuracy is the expected accuracy in the out-of-sample data set (i.e. original testing data set). Thus, the expected value of the out-of-sample error will correspond to the expected number of misclassified observations/total observations in the Test data set, which is the quantity: 1-accuracy found from the cross-validation data set.

Our outcome variable “classe” is a factor variable. We split the Training dataset into TrainTrainingSet and TestTrainingSet datasets.

```{r cache=TRUE, warning=FALSE, results='hide', cache.lazy=FALSE, message=FALSE}

#Loading the required packages
library(lattice)
library(ggplot2)
library(caret)
library(randomForest)
library(rpart)
library(rpart.plot)

set.seed(1234)

# Downloading the data
# Download training data set
url <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
file <- "pml-training.csv"
download.file(url, destfile = file, method = "curl")

# Download test data set
url <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
file <- "pml-testing.csv"
download.file(url, destfile = file, method = "curl")

# Loading and cleaning the data
trainingset <- read.csv("pml-training.csv", na.strings = c("NA","#DIV/0!", ""))
testingset <- read.csv("pml-testing.csv", na.strings = c("NA","#DIV/0!", ""))

# Delete columns with all missing values
trainingset<-trainingset[,colSums(is.na(trainingset)) == 0]
testingset <-testingset[,colSums(is.na(testingset)) == 0]

# Delete variables are irrelevant to our current project: user_name, raw_timestamp_part_1, raw_timestamp_part_,2 cvtd_timestamp, new_window, and  num_window (columns 1 to 7). 
trainingset <- trainingset[,-c(1:7)]
testingset <- testingset[,-c(1:7)]

# Partition the data so that 75% of the training dataset into training and the remaining 25% to testing
traintrainset <- createDataPartition(y=trainingset$classe, p=0.75, list=FALSE)
TrainTrainingSet <- trainingset[traintrainset, ] 
TestTrainingSet <- trainingset[-traintrainset, ]

# The variable "classe" contains 5 levels: A, B, C, D and E. A plot of the outcome variable will allow us to see the frequency of each levels in the TrainTrainingSet data set and # compare one another.

plot(TrainTrainingSet$classe, col = "blue", main = "Plot of levels of variable classe within the TrainTrainingSet data set", xlab = "classe", ylab = "Frequency")
```

### Prediction Model 1: Decision Tree

```{r cache = TRUE}
model1 <- rpart(classe ~ ., data=TrainTrainingSet, method="class")

prediction1 <- predict(model1, TestTrainingSet, type = "class")

# Plot the Decision Tree
rpart.plot(model1, main="Classification Tree", extra=102, under=TRUE, faclen=0)

# Test results on our TestTrainingSet data set:
confusionMatrix(prediction1, TestTrainingSet$classe)
```

***

### Prediction Model 2: Random Forest

```{r cache = TRUE}
model2 <- randomForest(classe ~. ,data=TrainTrainingSet, method = "class")

prediction2 <- predict(model2, TestTrainingSet, type = "class")

# Test results on TestTrainingSet data set:
confusionMatrix(prediction2, TestTrainingSet$classe)
```

***

### Decision on which Model to Use

Random Forest algorithm performed better than Decision Tree. Accuracy for Random Forest model was 0.995 (95% CI: (0.993, 0.997)) compared to Decision Tree model with 0.739 (95% CI: (0.727, 0.752)). The Random Forest model is chosen. The expected out-of-sample error is estimated at 0.005, or 0.5%.

***

### Submission

Here is the final outcome based on the Prediction Model 2 (Random Forest) applied against the Testing dataset

```{r}
# Predict outcome levels on the original Testing data set using Random Forest algorithm
predictfinal <- predict(model2, testingset, type="class")
predictfinal
```
