####Overview (copied from Project Instructions)
One of the most exciting areas in all of data science right now is wearable computing. Companies like Fitbit, Nike, and Jawbone Up are racing to develop the most advanced algorithms to attract new users. The data linked to from the course website represent data collected from the accelerometers from the Samsung Galaxy S smartphone. 

####Requirements
You should create one R script called run_analysis.R that does the following.

1.  Merges the training and the test sets to create one data set
2.  Extracts only the measurements on the mean and standard deviation for each measurement
3.  Uses descriptive activity names to name the activities in the data set
4.  Appropriately labels the data set with descriptive variable names
5.  From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject

####Description of the data
http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones 

####Data for the project
https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip 

####Process
1.  Load the raw data into R datasets
2.  Use data in the description tables to add column names to the data tables
3.  Use cbind to combine the Subjects, the Labels and the data for both the Training and Test data
4.  Merge the Label Id to pull in the description of the activities
5.  Merge the Training and Test datasets
6.  Extract the column names from the final table to determine relevant columns
7.  Identify the relevant columns (columns containing "mean()" or "std()"
8.  Extract the relevant columns and create the Final dataset
9.  Cleanse the column names.  i.e. remove special characters
10.  Select the columns to summarize by
11.  Summarize the table and calculate the mean for each metric
12.  Write the table to a text file

####Load the activity and features data into R data frames

```
activityLabel <- read.table("./data/UCI HAR Dataset/activity_labels.txt",
        col.names = c("LabelId","Activity"))

features <- read.table("./data/UCI HAR Dataset/features.txt",
        col.names = c("FeatureId","Feature"))
```

####Load the Training data into R data frames and set the column names in the Training data frame based on the data from the features data

```
TrainSet <- read.table("./data/UCI HAR Dataset/train/x_train.txt")
colnames(TrainSet) <- features[,2]

TrainLabels <- read.table("./data/UCI HAR Dataset/train/y_train.txt",
        col.names = c("LabelId"))

TrainSubjects <- read.table("./data/UCI HAR Dataset/train/subject_train.txt",
        col.names = c("SubjectId"))
```

####Merge the Training data into a single data frame

```
TrainTemp <- cbind(TrainSubjects, TrainLabels, TrainSet)
FullTrain <- merge(activityLabel, TrainTemp, 
                   by.x="LabelId", 
                   by.y="LabelId", 
                   all=TRUE)
```

####Load the Test data into R data frames and set the column names in the Test data frame based on the data from the features data

```
TestSet <- read.table("./data/UCI HAR Dataset/test/x_test.txt")
colnames(TestSet) <- features[,2]

TestLabels <- read.table("./data/UCI HAR Dataset/test/y_test.txt",
        col.names = c("LabelId"))

TestSubjects <- read.table("./data/UCI HAR Dataset/test/subject_test.txt",
        col.names = c("SubjectId"))
```

####Merge the Test data into a single data frame

```
TestTemp <- cbind(TestSubjects, TestLabels, TestSet)
FullTest <- merge(activityLabel, TestTemp, 
                   by.x="LabelId", 
                   by.y="LabelId", 
                   all=TRUE)
```

####Merge the Training and Test data frames into a single data frame

```
FullDataset <- merge(FullTest, FullTrain, all=TRUE)
```

####Extract column names from the data frame

```
FullDatasetColNames <- colnames(FullDataset)
```

####Identify the columns that contain the Mean and Standard Deviations

```
meanColumns <- grep("mean()",FullDatasetColNames, fixed=TRUE)
stdColumns  <- grep("std()",FullDatasetColNames, fixed=TRUE)
relevantColumns <- c(2, 3, meanColumns, stdColumns)
```

####Load the dplyr package

```
library(dplyr)
```

####Select only the relevant columns out of the full data frame

```
FinalDataset <- select(FullDataset, relevantColumns)
```

####Adjust the column names to remove special characters

```
BadColumnNames <- colnames(FinalDataset)
NewColumnNames <- gsub("\\()","",BadColumnNames)
NewColumnNames <- gsub("-","",NewColumnNames)
```

####Set the column names in the Final Dataset

```
colnames(FinalDataset) <- NewColumnNames
```

####create a list with Activity and Subject for grouping

```
grpColumns <- names(FinalDataset)[1:2]
dots <- lapply(grpColumns, as.symbol)
```

####Create the final summarized dataset grouped by Activity and Subject

```
summaryDataset <-
FinalDataset %>%
        group_by_(.dots=dots, add=TRUE) %>%
        summarise_each(funs(mean))
```

####Write the summarized dataset out to a txt file

```
write.table(summaryDataset, file = "./data/final.txt", sep=",", row.names=FALSE)
```
