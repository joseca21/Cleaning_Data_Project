**This is a README file which explains how the code to consolidate and clean the Samsung dataset works.**

First of all, it assumes the 'UCI HAR Dataset' folder is set as working directory in R (using setwd() to do this)


##### QUESTION 1: Merges the training and the test sets to create one data set

###### A: Reading and preparing test data.

*Step 1: Read Test set from 'test/X_test.txt' and assign data to data.frame Xtest_1 (with no headers)*

Xtest_1 <- read.csv("./test/X_test.txt", header=F, sep="", comment.char ="", colClasses="numeric")

*Step 2: Read List of all features (headers for Test/Train data) from 'features.txt' and assign data to data.frame called Header_Train_Test (with no headers)*

Header_Train_Test <- read.csv("./features.txt", header=F, sep="")

*Step 3: Transpose 2nd column -column $V2- in Header_Train_Test (containing headers) from columns to rows and assign transposed data to T_Header_Train_Set*

T_Header_Train_Set <- t(Header_Train_Test$V2)

*Step 4: Add transposed headers (i.e. T_Header_Train_Set) to Test set (i.e. Xtest_1)*

colnames(Xtest_1) <- T_Header_Train_Set

*Step 5: Read Test labels containing the activity IDs for Test set from 'test/y_test.txt' and assign data to data.frame ytest_1 (excluding headers)*

ytest_1 <- read.csv("./test/y_test.txt", header=F, sep="", comment.char ="", colClasses="numeric")

*Step 6: Read subject IDs containing subjects who peformed activities measured in Test set from 'test/subject_test.txt' and assign data to data.frame subjecttest_1 (with no headers)*

subjecttest_1 <- read.csv("./test/subject_test.txt", header=F, sep="", comment.char ="", colClasses="numeric")

*Step 7: Merge Subject IDs (subjecttest_1) with Activity IDs (ytest_1) and assign merged data to test_subj_activ. Then insert descriptive headers to data.frame*

test_subj_activ <- cbind(subjecttest_1, ytest_1)

colnames(test_subj_activ) <- c("subjectID","activityID")

*Step 8: Merge dataset containing both Activity and subject IDs (test_subj_activ) with Test set (Xtest_1)*

Xtest_2 <- cbind(test_subj_activ,Xtest_1)



###### B: Reading and preparing training data.

*Step 1: Read Training set from 'train/X_train.txt' and assign data to data.frame Xtrain_1 (with no headers)*

Xtrain_1 <- read.csv("./train/X_train.txt", header=F, sep="", comment.char ="", colClasses="numeric")

*Step 2: Add transposed headers (i.e. previously saved to T_Header_Train_Set) to Training set (i.e. Xtrain_1)*

colnames(Xtrain_1) <- T_Header_Train_Set

*Step 3: Read Test labels containing the activity IDs for Training set from 'train/y_train.txt' and assign data to data.frame ytrain_1 (with no headers)*

ytrain_1 <- read.csv("./train/y_train.txt", header=F, sep="", comment.char ="", colClasses="numeric")

*Step 4: Read subject IDs containing subjects who peformed activities measured in Train set from 'train/subject_train.txt' and assign data to data.frame subjecttrain_1 (with no headers)*

subjecttrain_1 <- read.csv("./train/subject_train.txt", header=F, sep="", comment.char ="", colClasses="numeric")

*Step 5: Merge Subject IDs (subjecttrain_1) with Activity IDs (ytrain_1) and assign merged data to train_subj_activ. Then insert descriptive headers to data.frame*

train_subj_activ <- cbind(subjecttrain_1, ytrain_1)

colnames(train_subj_activ) <- c("subjectID","activityID")

*Step 6: Merge dataset containing both Activity and subject IDs (train_subj_activ) with Train set (Xtrain_1)*

Xtrain_2 <- cbind(train_subj_activ,Xtrain_1)



###### C: Merge Test and Training Data Sets into a Master Data set called MergedData

MergedData <- rbind(Xtest_2, Xtrain_2)




##### QUESTION 2: Extracts only the measurements on the mean and standard deviation for each measurement.
*Include any field header which contains either "mean(" or "std(" or "subject" or "activity"
(last 2 options to include activityID and subjectID fields)
Once all relevant fields are extracted (using the Mean_Std variable), a data.frame is created with the 68 fields selected and assigned to MergedData2*

Mean_Std <- grepl("subject", names(MergedData), fixed=TRUE) | grepl("activity", names(MergedData), fixed=TRUE) | grepl("mean(", names(MergedData), fixed=TRUE) | grepl("std(", names(MergedData), fixed=TRUE)

MergedData2 <- MergedData[, Mean_Std]



##### QUESTION 3: Uses descriptive activity names to name the activities in the data set

*Activity names are read from activity_labels.txt and assigned to Activity_Labels data.frame.
Descriptive names are then added to Activity_Labels dataset*

Activity_Labels <- read.csv("./activity_labels.txt", header=F, sep="")

colnames(Activity_Labels) <- c("activityID","activityName")

*Activity Names is then merged to Master Dataset (MergedData2) and a new master dataset (now containing 69 features) is assigned to MergedData3 dataset*

MergedData3 <- merge(Activity_Labels,MergedData2,by = "activityID")

*FIelds are then re-arranged in MergedData3 master dataset and ActivityID field is removed*
*New output is assigned to MergedData4 data.frame*

New MergedData4 data.frame has now 68 fields (as ActivityID was removed)

MergedData4 <- MergedData3[ , c(3,2,4:69)]




##### QUESTION 4: Appropriately labels the data set with descriptive variable names

*Brackets () are removed from all measurement headers to tidy up data labels.*
*The headers naming convention used for signals is as follows: [time/frequency domain signal].[mean/std].[axial raw XYZ - if applicable]*

names(MergedData4) <- gsub("\\(\\)", "", names(MergedData4)) [Note escape character is double '\' but .md file doesn't seem to recognise it]


##### QUESTION 5: Creates a second, independent tidy data set with the average of each variable for each activity and each subject

*Aggregated output is then assigned to Agg_MergedData4 data.frame*

Agg_MergedData4 <- aggregate(MergedData4[ , c(3:68)], by=list(factor(MergedData4$subjectID),MergedData4$activityName), FUN="mean", na.rm=TRUE)

*As the aggregate function changes the names of the 2 fields used for aggregation to Group.n, original tidy names are replaced*

names(Agg_MergedData4)[1] <- "Subject ID"

names(Agg_MergedData4)[2] <- "Activity Name"

**Final output from question 5 is then exported to txt. fileand saved in root directory**
**Exports data to a comma delimited text file**

#### This is the tidy dataset uploaded for this assignment

write.table(Agg_MergedData4,file = "Aggregated_Merged_TidyData_GalaxyS.txt")
