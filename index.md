# Machine Learning Course Project
Phillip  
March 14, 2016  



The following report was an assignment completed during my enrollment in the John Hopkin's Machine Learning Course on Coursera.  The Project Goal and Background Information sections below are copied directly from the course website while the rest of the paper details how I went about completing the assignment. 



###Project Goal
The goal of your project is to predict the manner in which they did the exercise. This is the “classe” variable in the training set. You may use any of the other variables to predict with. You should create a report describing how you built your model, how you used cross validation, what you think the expected out of sample error is, and why you made the choices you did. You will also use your prediction model to predict 20 different test cases.

Your submission should consist of a link to a Github repo with your R markdown and compiled HTML file describing your analysis. Please constrain the text of the writeup to < 2000 words and the number of figures to be less than 5. It will make it easier for the graders if you submit a repo with a gh-pages branch so the HTML page can be viewed online (and you always want to make it easy on graders :-).

You should also apply your machine learning algorithm to the 20 test cases available in the test data above. Please submit your predictions in appropriate format to the programming assignment for automated grading. See the programming assignment for additional details.



###Background Information
Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. 

In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).



###Getting and Loading Datasets 
The following anaylsis was conducted using R version 3.1.3 (2015-03-09) and a MacBook Pro (Retina, 15-inch, Mid 2014) with a 2.2 Ghz Intel Core i7 Processor and 16 GB 1600 MHz DDR3 Memory.  

The 'caret' and 'radomForest' packages will be used for this project so they are loaded into the working environment first.  I've also set a seed (1127) to be used for pseudo random number generation so others are able to reproduce the model below.  

The datasets were then downloaded from the websites listed above and stored in memory.  Alternatively, the datasets could have been downloaded to the hard drive and read in from there, however I chose not to do so for this project.  Since the test dataset is being used for course grading purposes the training dataset will need to be split 60/40 into a testing and training dataset to build the model.  After the data is split I'll print out the dimensions of both sets to review them.  
 
 

```r
library(caret)
```

```
## Loading required package: lattice
## Loading required package: ggplot2
```

```r
library(randomForest)
```

```
## randomForest 4.6-12
## Type rfNews() to see new features/changes/bug fixes.
```

```r
set.seed(1127)

inTrainURL <- "http://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
testURL <- "http://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"

trainData <- read.csv(url(inTrainURL), na.strings=c("NA","#DIV/0!",""))
courseTesting <- read.csv(url(testURL), na.strings=c("NA","#DIV/0!",""))

inTrain <- createDataPartition(y=trainData$classe, p=0.6, list=FALSE)
training <- trainData[inTrain,]
testing <- trainData[-inTrain,]
dim(training); dim(testing)
```

```
## [1] 11776   160
```

```
## [1] 7846  160
```

```r
# rm(inTrain,trainData)
```


###Data Cleaning
As you can see above, the datasets contain 160 variables (less the one prediction variable) so before the model is created I'll want remove any of those which may have little or no impact.  I'll start by first looking at the number of na values per variable across the dataset to see if anything stands out.    



```r
NAcounts <- apply(training, 2, function(x) length(which(!is.na(x))))
table(NAcounts)
```

```
## NAcounts
##     0   197   208   210   232   243   244   246   249   250 11776 
##     6     7     2     2     2     2     5     5     2    67    60
```

The table above shows there are 60 variables which have values for every record, while the rest of the variables have far less (i.e. a lot of na's), so I'll drop those across all three datasets. Next, I'll remove any variables that have near zero variance.  To do this I'm wrapping the function in an IF statement because if it returns zero variables with near zero variance the function will remove all of the variables from the datasets. 



```r
training <- training[ , colSums(is.na(training)) == 0]
if (length(nearZeroVar(training)) > 0) {
  training <- training[, -nearZeroVar(training)] 
}

testing <- testing[ ,colSums(is.na(testing)) ==0]
if (length(nearZeroVar(testing)) > 0) {
  testing <- testing[, -nearZeroVar(testing)] 
}

courseTesting <- courseTesting[ ,colSums(is.na(courseTesting)) ==0]
if (length(nearZeroVar(courseTesting)) > 0) {
  courseTesting <- courseTesting[, -nearZeroVar(courseTesting)] 
}
dim(training); dim(testing)
```

```
## [1] 11776    59
```

```
## [1] 7846   59
```

With 59 variables still in the datasets that means there was only one variable which had a near zero variance.  One last thing that I can do is look at the names of the variables and possible the raw data to determine if any others can be dropped.  This is usually something that would be done initially, however I was not able to find any documents which explained what each of the variables were and with 160 to go through I skipped that step.  



```r
names(training)
```

```
##  [1] "X"                    "user_name"            "raw_timestamp_part_1"
##  [4] "raw_timestamp_part_2" "cvtd_timestamp"       "num_window"          
##  [7] "roll_belt"            "pitch_belt"           "yaw_belt"            
## [10] "total_accel_belt"     "gyros_belt_x"         "gyros_belt_y"        
## [13] "gyros_belt_z"         "accel_belt_x"         "accel_belt_y"        
## [16] "accel_belt_z"         "magnet_belt_x"        "magnet_belt_y"       
## [19] "magnet_belt_z"        "roll_arm"             "pitch_arm"           
## [22] "yaw_arm"              "total_accel_arm"      "gyros_arm_x"         
## [25] "gyros_arm_y"          "gyros_arm_z"          "accel_arm_x"         
## [28] "accel_arm_y"          "accel_arm_z"          "magnet_arm_x"        
## [31] "magnet_arm_y"         "magnet_arm_z"         "roll_dumbbell"       
## [34] "pitch_dumbbell"       "yaw_dumbbell"         "total_accel_dumbbell"
## [37] "gyros_dumbbell_x"     "gyros_dumbbell_y"     "gyros_dumbbell_z"    
## [40] "accel_dumbbell_x"     "accel_dumbbell_y"     "accel_dumbbell_z"    
## [43] "magnet_dumbbell_x"    "magnet_dumbbell_y"    "magnet_dumbbell_z"   
## [46] "roll_forearm"         "pitch_forearm"        "yaw_forearm"         
## [49] "total_accel_forearm"  "gyros_forearm_x"      "gyros_forearm_y"     
## [52] "gyros_forearm_z"      "accel_forearm_x"      "accel_forearm_y"     
## [55] "accel_forearm_z"      "magnet_forearm_x"     "magnet_forearm_y"    
## [58] "magnet_forearm_z"     "classe"
```


After reviewing the names and looking at the some of the raw data I've determine that the first six variables can be removed.  The first variable just contains row number, the second is the users name, and three through six contain data related to a timestamp.



```r
training <- training[, -c(1:6)]  
testing <- testing[ ,-c(1:6)]
courseTesting <- courseTesting[ ,-c(1:6)]
```


###Model Training
The model is then created using the randomForest package with the number of tress set to the default value of 500.  Once done it's used to predict the classe variable in the testing dataset for cross validation purposes, where we'll look at the accuracy percentage to determine how well it performed.  



```r
fitMod <- randomForest(classe ~ .,training, ntree=500)
testing$predicted.response <- predict(fitMod, testing)
confusionMatrix(data=testing$predicted.response, reference=testing$classe, positive='yes')
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 2230    7    0    0    0
##          B    2 1505   10    0    0
##          C    0    5 1355   16    2
##          D    0    1    3 1269    3
##          E    0    0    0    1 1437
## 
## Overall Statistics
##                                           
##                Accuracy : 0.9936          
##                  95% CI : (0.9916, 0.9953)
##     No Information Rate : 0.2845          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.9919          
##  Mcnemar's Test P-Value : NA              
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9991   0.9914   0.9905   0.9868   0.9965
## Specificity            0.9988   0.9981   0.9964   0.9989   0.9998
## Pos Pred Value         0.9969   0.9921   0.9833   0.9945   0.9993
## Neg Pred Value         0.9996   0.9979   0.9980   0.9974   0.9992
## Prevalence             0.2845   0.1935   0.1744   0.1639   0.1838
## Detection Rate         0.2842   0.1918   0.1727   0.1617   0.1832
## Detection Prevalence   0.2851   0.1933   0.1756   0.1626   0.1833
## Balanced Accuracy      0.9989   0.9948   0.9935   0.9929   0.9982
```


Wow, 99.3% accuracy!  With an accuracy this high the out of sample error rate should be pretty low but let's double check to make sure. 



```r
missClass = function(values, prediction) {
  sum(prediction != values)/length(values)
}
missClass(testing$classe, testing$predicted.response)
```

```
## [1] 0.006372674
```

Based on the miss classification rate on the testing subset, an unbiased estimate of the random forest's out-of-sample error rate is 0.06%.


###Quiz Prediction

```r
predict(fitMod, courseTesting)
```

```
##  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
##  B  A  B  A  A  E  D  D  A  A  B  C  B  A  E  E  A  B  B  B 
## Levels: A B C D E
```
