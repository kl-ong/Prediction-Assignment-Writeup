# Prediction Assignment Writeup


```r
library(caret)
```

```
## Warning: package 'caret' was built under R version 3.3.2
```

```
## Loading required package: lattice
```

```
## Loading required package: ggplot2
```

```
## Warning: package 'ggplot2' was built under R version 3.3.2
```

##### Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. 

##### In this project, the goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. 

##### Participants were asked to perform one set of 10 repetitions of the Unilateral Dumbbell Biceps Curl in five different fashions:  
* Class A: exactly according to the specification
* Class B: throwing the elbows to the front
* Class C: lifting the dumbbell only halfway
* Class D: lowering the dumbbell only halfway
* Class E: throwing the hips to the front  

##### Class A corresponds to the specified execution of the exercise, while the other 4 classes correspond to common mistakes


#### Data Loading 

```r
#Data Load
rawTrain <- read.csv("pml-training.csv")
rawTest <- read.csv("pml-testing.csv")
```

#### Data Cleansing 
Removing the near zero variance and rows with NAs

```r
#Remove columns with NAs
testing <- rawTest[, colSums(is.na(rawTrain))==0]
training <- rawTrain[, colSums(is.na(rawTrain))==0]


#Remove near zero variance 
x = nearZeroVar(training, saveMetrics = TRUE)
nzv<-x[,"nzv"] 
training <- training[,!nzv]
testing <- testing[,!nzv]

#Remove columns of user identity and time stamp data
colsToBeRemoved <- grepl("^X|user|timestamp|window", names(training))
training <- training[, !colsToBeRemoved]
colsToBeRemoved <- grepl("^X|user|timestamp|window", names(testing))
testing <- testing[, !colsToBeRemoved]

#Remove problem_id column
colsToBeRemoved <- grepl("^problem_id", names(testing))
testing <- testing[, !colsToBeRemoved]
```


#### Data Modeling
Using random forests for modelling and predicting as it is the most accurate

```r
set.seed(33123)

controlRf <- trainControl(method="cv", 5)
modelRf <- train(classe ~ ., data=training, method="rf", trControl=controlRf, ntree=250)
```

```
## Loading required package: randomForest
```

```
## Warning: package 'randomForest' was built under R version 3.3.2
```

```
## randomForest 4.6-12
```

```
## Type rfNews() to see new features/changes/bug fixes.
```

```
## 
## Attaching package: 'randomForest'
```

```
## The following object is masked from 'package:ggplot2':
## 
##     margin
```

```r
modelRf
```

```
## Random Forest 
## 
## 19622 samples
##    52 predictor
##     5 classes: 'A', 'B', 'C', 'D', 'E' 
## 
## No pre-processing
## Resampling: Cross-Validated (5 fold) 
## Summary of sample sizes: 15696, 15698, 15698, 15698, 15698 
## Resampling results across tuning parameters:
## 
##   mtry  Accuracy   Kappa    
##    2    0.9943429  0.9928440
##   27    0.9941392  0.9925863
##   52    0.9890432  0.9861399
## 
## Accuracy was used to select the optimal model using  the largest value.
## The final value used for the model was mtry = 2.
```
With a 5-fold cross validation, the accuracy achieved is about 99.4%.

#### Predicting test data with the random forests model
Thus we use random forests model that we have gather for predicting the test data.

```r
predRf <- predict(modelRf, testing)
predRf
```

```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```


###### More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).

###### Data is courtesy http://groupware.les.inf.puc-rio.br/har 

