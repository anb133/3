Set working directory

    dir.create("Project")
    setwd("./Project")

Load required packages

    library(data.table)
    library(plyr)
  
##1. Merge the training and the test sets to create one data set
####Get the Data 
Step1. Download the file

    fileUrl<- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
    download.file(fileUrl, destfile="./Dataset.zip", mode="wb")

Step2. Unzip dataset in a /Project directory and list all files

    unzip(zipfile="./Dataset.zip")
    list.files("UCI HAR Dataset", recursive=TRUE)

####Read Activity, Subject and Feature files
First, read Activity, Subject and Features data for Train: 

    get_data<- {setwd("./UCI HAR Dataset/train")
            dActivityTrain<- read.table("./y_train.txt", header=FALSE)
            dSubjectTrain<- read.table("./subject_train.txt", header= FALSE)
            dFeaturesTrain<- read.table("./X_train.txt", header=FALSE)          
            setwd("../")
After that read the same data for Test:

            setwd("./test")
            dActivityTest <- read.table("./y_test.txt", header=FALSE)
            dSubjectTest <- read.table("./subject_test.txt", header=FALSE)
            dFeaturesTest<- read.table("./X_test.txt", header=FALSE) 
            setwd("../")
            setwd("../")}

####Join data tables by rows
    dSubject<- rbind(dSubjectTrain, dSubjectTest)
    dActivity<- rbind(dActivityTrain, dActivityTest)
    dFeatures<- rbind(dFeaturesTrain, dFeaturesTest)

####Give names to the variables
    names(dSubject)<- "Subject"
    names(dActivity)<- "Activity"

    dFeaturesNames <- read.table(("./UCI HAR Dataset/features.txt"),head=FALSE)
    head(dFeaturesNames)
    names(dFeatures) <- dFeaturesNames$V2 

####Merge tables by columns to create one data set (Data) 
    SubjectActivity<- cbind(dSubject, dActivity)
    Data<- cbind(dFeatures, SubjectActivity)
    str(Data)

##(2) Extract only the measurements on the mean and standard deviation for each measurement

####Subset "Name of Features" by measurements on the mean and standard deviation
    sdFeaturesNames<-dFeaturesNames$V2[grep("mean\\(\\)|std\\(\\)", dFeaturesNames$V2)]

####Subset data set by selected "Names of Features"
    selectedNames<-c(as.character(sdFeaturesNames), "Subject", "Activity" )
    Data<-subset(Data,select=selectedNames)


##(3) Use descriptive activity names to name the activities in the data set

####Read activity names and set new column names in the obtained dataset
    activityNames<- read.table("./UCI HAR Dataset/activity_labels.txt")
    names(activityNames)<- c("Activity", "ActivityName")

####Factorize the Activity variable 
    activity<- factor(Data$Activity,
                   levels=c(1:6),
                   labels= activityNames$ActivityName)
    Data$Activity<- activity

####Check the result
    head(Data$Activity, 40)

##(4) Appropriately label the data set with descriptive variable names

####Check initial names
    names(Data)

####Replace the following:
t -> time

f -> frequency

Acc -> Accelerometer

Gyro -> Gyroscope

Mag -> Magnitude

BodyBody -> Body

std -> SD

mean -> Mean 

    names(Data)<- gsub("^t", "time", names(Data))
    names(Data)<- gsub("^f", "frequency", names(Data))
    names(Data)<- gsub("Acc", "Accelerometer", names(Data))
    names(Data)<- gsub("Gyro", "Gyroscope", names(Data))
    names(Data)<- gsub("Mag", "Magnitude", names(Data))
    names(Data)<- gsub("BodyBody", "Body", names(Data))
    names(Data)<- gsub("std", "SD", names(Data))
    names(Data)<- gsub("mean", "Mean", names(Data))

####Check new names
names(Data)


## (5) Create a second, independent tidy data set with the average of each variable for each activity and each subject
		Data2<-aggregate(. ~Subject + Activity, Data, mean) 
		Data2

This data set a is set of variables for each activity and subject. As there are 30 subjects and 6 ativities, initial 10299 observations were split into 180 groups. The data set provides average mean and standard deviation features for each group. 
