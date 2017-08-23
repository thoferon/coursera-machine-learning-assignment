# Predicting physicial activity
Thomas Feron  
18 August 2017  



## Data 


```r
training <- read.csv("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv", na.strings = c("NA", ""))
unknown  <- read.csv("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv",  na.strings = c("NA", ""))
```

A lot of the rows have missing values but they are located in a few columns with a lot of missing values. Let's just get rid of those columns as they would not be very useful for predictions considering how many values are missing.


```r
training <- training[,sapply(training, function(col) { sum(is.na(col)) == 0 })]
training <- training[,-c(1,3,4,5,6,7,8)]
unknown <- unknown[,c(names(training)[-53], "problem_id")]

inClass <- createDataPartition(training$classe, p = 0.6, list = FALSE)
testing  <- training[-inClass,]
training <- training[inClass,]
```

## Model

We now train a random forest model on this tidy data set using K-fold cross validation (k = 5) to improve on the accuracy. Random forest will try to split the data set on some feature repeatedly to classify the data points. This seems to be a good solution to the problem at hand and, as we shall see, the resulting accuracy is quite good.


```r
library(caret)
set.seed(31415)
ctrl <- trainControl(method = "cv", number = 5)
fit <- train(classe ~ ., method = "rf", data = training, trControl = ctrl)
```

## Out-of-sample error

To estimate the out-of-sample error, let's calculate the accuracy on the testing set.


```r
preds <- predict(fit, newdata = testing)
confusionMatrix(preds, testing$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 2231   15    0    0    0
##          B    0 1497   16    0    0
##          C    0    5 1345   16    3
##          D    0    1    7 1267    7
##          E    1    0    0    3 1432
## 
## Overall Statistics
##                                           
##                Accuracy : 0.9906          
##                  95% CI : (0.9882, 0.9926)
##     No Information Rate : 0.2845          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.9881          
##  Mcnemar's Test P-Value : NA              
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9996   0.9862   0.9832   0.9852   0.9931
## Specificity            0.9973   0.9975   0.9963   0.9977   0.9994
## Pos Pred Value         0.9933   0.9894   0.9825   0.9883   0.9972
## Neg Pred Value         0.9998   0.9967   0.9964   0.9971   0.9984
## Prevalence             0.2845   0.1935   0.1744   0.1639   0.1838
## Detection Rate         0.2843   0.1908   0.1714   0.1615   0.1825
## Detection Prevalence   0.2863   0.1928   0.1745   0.1634   0.1830
## Balanced Accuracy      0.9984   0.9918   0.9897   0.9915   0.9962
```



We see the accuracy is of 99.06% so the estimated error rate is 0.94%.

## Predictions

We now use our model to predict the classe of the unknown data set.


```r
predict(fit, newdata = unknown)
```

```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```
