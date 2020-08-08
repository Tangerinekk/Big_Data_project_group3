# project-group-3
### Group member: <br/>
Qi Gui<br/> Yichen Li<br/>   Junzhe Yin<br/> Jiatian Ye<br/>

### Executive Summary<br/>
In the project, we firstly use spark and s3 bucket to prepare, clean and sort the data of the hard drive from Backblaze data center. After that, we select certain columns as we mentioned in our project proposal to do exploratory analysis, and find out the trend of each column in recent 2 year(2018-2019). What’s more, we also built a model to calculate the lifespan of a failed hard drive. Then, the most important part 3, we build 3 types of model: “Logistic Model”, “Random Forest Classification”, and “GBT Classifier”, test our model, and compare their accuracy. Finally, we draw the conclusions about what we did and plan about future work.<br/>

### Introduction<br/>

#### Dataset Source<br/>
The data is from Backblaze data center, which is a data storage provider.Backblaze started publishing hard drive data in its B2 Cloud Storage product since 2013.<br/>

The data is about a snapshot of each operational hard drive.This snapshot includes basic drive information along with the S.M.A.R.T. statistics reported by that drive. The daily snapshot of one drive is one record or row of data. All of the drive snapshots for a given day are collected into a file consisting of a row for each active hard drive. <br/>

Data link:https://www.backblaze.com/b2/hard-drive-test-data.html

#### Initial plan<br/>
Our initial plan is to analyze diverse failures based on shared a lot of Backblaze data about hard drive failure statistics. In general, we prefer to predict drive failures, and use the hard drives’ built-in SMART metrics to help, in order to identify which SMART stats are good predictors of drive failure below. Then, we could compare the failure of certain disk drive with the market report to see if there is some inconstant. <br/>
In addition, grouping disk drive in different Series number model or manufacture, we could statistic the failure rate and what is the main reason of failure.<br/>

### Part 1:Data preparation <br/>  
#### Data prepare process:<br/>
Our original dataset is separated into 8 different csv files (2018 Q1, Q2, Q3, Q4, and 2019Q1, Q2, Q3, Q4), which is hard to analyze as a whole. Therefore, we decide to combine them into one single csv file for further use. To combine the files, we first checked the numbers of rows and columns for all the eight csv files, and found 2018 Q1 has the least column numbers, so we dropped off the excess columns of the rest 7 files to let all of 8 files have the same columns. Then, since the combined data is too large (over 30GB), we decide to change the format of the file from csv to parquet to save loading time, and the parquet format is saved in our s3 bucket by hadoop. (s3://bigdata-group3/mergedata/hardrive.parquet) <br/>

#### Data clean process:<br/>
Before formal analysis, we clean our data by the following steps:<br/>
1. Find out the columns that have over 50% missing value and drop them. The rest good columns number is 45.<br/>
2. Drop the null value in each rows based on what we got in the 1 step, the row numbers decrease from 76,714,860 to 57364274.<br/>


### Part 2:Exploratory Analysis Section(EDA) <br/>
  In our exploratory analysis section, we imported the following three python packages: Pandas, Pyplot, and pyspark.sql. First, we counted the total number of failed drives and good drives using the spark.sql function. The output shows there are total of 3,951 failed drives and 76,711,269 good drives. Secondly, we also tried to summarize the capacity of drives also using the spark.sql function. The results show the max capacity among drives is 16000900661248 bytes, and the average capacity for these drives is 7.554529303944754*E^12 bytes. We did the same process again but this time, we distinguished between good drives and failed drives. We found that the maximum, minimum, and the average capacities have no big difference between good and failed drives.<br/>
  <br/>
   Next, we looked at the S.M.A.R.T. value (different value indicates different features of the drives) of drives. We found that S.M.A.R.T. 5, 187, 188, 197, and 198 have big difference between good drives and drives with failures. The results are: 
1.	Drives with failures have much higher reallocate_ sector count (S.M.A.R.T. 5)
2.	Drives with failure = 0 have nearly 0 Reported_Uncorrectable_Errors (S.M.A.R.T. 187)
3.	Failure disk has larger timeout (S.M.A.R.T. 188)
4.	Failure drives have larger timeout (S.M.A.R.T. 197)
5.	Failure disks have higher marks (S.M.A.R.T. 198)

Then, to look specifically at these five S.M.A.R.T data, we start to visualize them with bar plot grouped by quarter, and here are conclusions:<br/>
1. The average Reported Uncorrectable Errors increases by quarter from 2018-2019.<br/>
2. The average reallocated Count also increases as Quarter increases.<br/>
3. The Command Time out also increases quarterly from 2018 to 2019, however, it increases at a lower rate compared to SMART5 and SMART187.<br/>
4. The average current count of unstable sectors has maximum value at 2018 Q2, and decreases until 2019Q2. It goes up from 2019Q2.<br/>
5. The average offline uncorrectable count by quarter has similar distribution as smart197, however, it does not increase too much from 2019Q3 to 2019Q4.<br/>

Finally, from the failure stats, we what to find out the lifetime of failure hard drive, in other words, calculating the lifespan of failure hard drive by day.<br/>
The reason we are interested in it is that both customers and merchants want to know the lifetime of a hard drive. On the one hand, for customers, they will know the minimum usage time of a hard drive; on the other hand, merchants could use lifetime as an indicator to check their product and re-enact the relevant insurance policies for commodities. According to our lifetime model and charts, we know the lifespan of a failed hard drive is 0-300 days, and the median usage time is around 170 days.<br/>
Note: all visualization parts are in this part<br/>

### Part 3；Building Model<br/>
#### Methods <br/>
We built three different models to find the accuracy of the data predicting the failure. They are “Logistic Model”, “Random Forest Classification”, and “GBT Classifier”. <br/>
We choose the “Logistic Model” because its application is very common to most data； we choose the “Random Forest Classification”, and “GBT Classifier” because they have good performance and fast operation during process.<br/>

#### Model Preparation:<br/>
After the  exploratory analysis of columns, we decided to build models to evaluate their classification accuracy. We will not include “date”, “model”, and “serial_number” columns in our data since they are string value and have no meaning explaining the failure. Then, we removed the failure from the “features” column since it is the dependent variable. Finally, to begin building our models, we split the dataset into training and testing sets, this will help us validate our model. <br/>

#### Model Results: <br/>
1.	Logistic Model: The logistic model has an accuracy of 85.37% when predict the disk failure.<br/>
2.	Random Forest Classification Model: It has 79.02% accuracy of predicting the failure.<br/>
3.	GBT Classifier: This model has a failure prediction accuracy of 74.32%.<br/>
Among the three classification models, the logistic regression performs the best when predict the disk failure.<br/>

### Challenges:<br/>
1. The data is too large, and it takes time for the model to adjust the parameters. <br/>
2. Too many null values in the original data cause us to delete a quarter of the data, which will affect the accuracy of the final model.<br/>
3. Due to the large size of data, the visualization part has many limitations, because pyspark and sql could not provide us with good plot function.<br/>

#### Overcome:<br/>
1. We work hard together and run different tests independently.<br/>
2. The accuracy is surely decreasing but the rest data is still large enough for us to explore.<br/>
3. We sort the data by certain groups, or we use toPandas() to use some plot functions on py. <br/>

### Future Work:<br/>
1. We will continue our work to improve the accuracy of the model by selecting different variables, and we will also try to adjust the parameters in our model.<br/>
2. Find relationships between variables, build more novel and interesting models, and visualize them.<br/>
3. Read articles related to the hard drive for further understanding our the data. <br/>

### Division of Labor:<br/>
Qi Gui : Data preparation, EDA<br/> 
Yichen Li : Data preparation, EDA<br/>
Junzhe Yin: Data preparation, building model<br/> 
Jiatian Ye； Data preparation, collect conclusion<br/>

### Takeaway from the course:<br/>
1. Learn how to use github and get more familiar with virtual machine(AWS).<br/>
2. Learn what is hadoop, mapreducer and how to use them, as well as improving the ability to use Spark and sql.<br/>
3. When we meet super large data as this time, we know how to process it.<br/>
