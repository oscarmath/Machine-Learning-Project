The goal of this project is to use data taken from accelerometers on the belt, forearm, arm, and dumbbell of 6 participants to predict how a participant is performing a task.  One way is correct and the other four are common incorrect ways. More information is available from the website here: <http://groupware.les.inf.puc-rio.br/har>. 

First we load the data.  I converted the `char` variables to `factor` after my caret protested.

```
library(caret)
library(dplyr)

# get the files' url and 
trainFileUrl <- "http://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
testFileUrl <- "http://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
download.file(trainFileUrl, "traindata.csv")
download.file(testFileUrl, "testdata.csv")
train_raw <- read.csv("traindata.csv", stringsAsFactors = FALSE)
test_raw <- read.csv("testdata.csv", stringsAsFactors = FALSE)
train_raw[sapply(train_raw, is.character)] <- lapply(train_raw[sapply(train_raw, is.character)], 
                                       as.factor)
test_raw[sapply(test_raw, is.character)] <- lapply(test_raw[sapply(test_raw, is.character)], 
                                       as.factor)
```

I removed all ID, bookkeeping, and statistical analysis embedded in the data with the following code.
```
train <- train_raw[-(1:7)]
train <- select(train, -contains("kurtosis"), -contains("skewness"), -starts_with("max"),
                -starts_with("min"), -starts_with("amplitude"), -starts_with("var"),
                -starts_with("avg"), -starts_with("stddev"))
```

In order to keep the time it takes to build the model on my computer, I took 10% of the data to build the model. I chose the random forest, `rf` method in caret, to build the model.
```
inTrain <- createDataPartition(y = train$classe, p=0.10, list=FALSE)
trainSmall <- train[inTrain,]
trainRest <- train[-inTrain,]
modFit <- train(classe ~ ., method="rf", data=trainSmall)
```

I cross-validated the model on the rest of the data.  The model was 95.2% accurate.
```

predictedtrain <- predict(modFit, newdata = trainRest)
missClass = function(values,prediction){sum((prediction != values)*1)/length(values)}
missClass(train$classe, predictedtrain)
```
