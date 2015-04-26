---
title: "Data Frame Manipulation"
author: "Oscar"
date: "04/26/2015"
---

This script takes information from to files, those files have the same number of variables, from which we are going to take a subset and merge the observations from both files, then apply the mean by two subgroups:

First, save all the file names into variables to make the code more readable
```{r}
testFile <- "./Data/UCI HAR Dataset/test/X_test.txt"
trainFile <- "./Data/UCI HAR Dataset/train/X_train.txt"
featureFile <- "./Data/UCI HAR Dataset/features.txt"
activityFile <- "./Data/UCI HAR Dataset/activity_labels.txt"

trainSubjectFile <- "./Data/UCI HAR Dataset/train/subject_train.txt"
trainActivityFile <- "./Data/UCI HAR Dataset/train/y_train.txt"
testSubjectFile <- "./Data/UCI HAR Dataset/test/subject_test.txt"
testActivityFile <- "./Data/UCI HAR Dataset/test/y_test.txt"
```

Then, read the train data, test data and the features that will become the column names
```{r}
testData <- read.csv(testFile, header = FALSE, sep = "")
trainData <- read.csv(trainFile, header = FALSE, sep = "")
features <- read.table(featureFile, header = FALSE, sep = "", colClasses = c("integer", "character"))
```

Assign the column names to the ´train´ and the ´test´ data
```{r}
colnames(trainData) <- features[,2]
colnames(testData) <- features[,2]
```

Find and subset the data Sets by columns related to mean and standard deviation only
```{r}
meanIndex<- grep("mean", features[,2])
stdIndex <- grep("std", features[,2])
mean_std_Index <- sort(c(meanIndex, stdIndex))

trainData <- trainData[,mean_std_Index]
testData <- testData[,mean_std_Index]
```
Next, read the activity labels and the ´Subjects´ and ´Activities´ related to ´train´ and ´test´ data sets
```{r}
activities <- read.table(activityFile, header = FALSE, sep = "", col.names = c("ANumber", "Activity"))

testActivities <- read.table(testActivityFile, header = FALSE, sep = "", col.names = "ANumber")
trainActivities <- read.table(trainActivityFile, header = FALSE, sep = "", col.names = "ANumber")

testSubjects <- read.table(testSubjectFile, header = FALSE, sep = "", col.names = "Subject")
trainSubjects <- read.table(trainSubjectFile, header = FALSE, sep = "", col.names = "Subject")
```

Next, add the Subjects and Activity related to each observation on each data set:
```{r}
trainActivities <- join(trainActivities, activities)
subDTrain <- cbind(trainSubjects, Activity = trainActivities[,2])
trainData <- cbind(subDTrain, trainData)

testActivities <- join(testActivities, activities)
subDTest <- cbind(testSubjects, Activity = testActivities[,2])
testData <- cbind(subDTest, testData)
```

Merge both data sets (train and test)
```{r}
dataSet <- rbind (trainData, testData)
```

And convert first column (Subject) to a factor, the second column (Activity) is already a factor
```{r}
dataSet$Subject <- factor(dataSet$Subject)
```

Next, split the data set by Subject and By Activity:
```{r}
dataSetSplit <- split(dataSet, list(dataSet$Subject, dataSet$Activity), drop = TRUE)
```

And calculate the mean of each column by Subject and by Activity
```{r}
dataSetMean <- t(sapply(dataSetSplit, function(x) colMeans(x[, 3:81])))
```

In order to add ´Subject´ and ´Activity´ corresponding to each row to the data set, this tricky script does the job: 
```{r}
uniqueFactors <- t(sapply(dataSetSplit, function(x) c (x[1,1], as.character(x[1,2]))))
dataSetMean <- cbind(data.frame(uniqueFactors), dataSetMean)
```

This is -hopefully- the result that the user want to see, finally write the data to a file:
```{r}
write.table(dataSetMean, "./Data/UCI HAR Dataset/train_test_mean.txt", quote=FALSE, row.names=FALSE, col.names=FALSE)
```

