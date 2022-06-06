# aws-cb-redshift-boilerplate : Session 1
This folder is the first session which provides a good starting point for Amazon Redshift by giving a complete end to end hands-on activity which you can try out on personal AWS account. Follow through the steps mentiooned below carefully to become master of Aamzon Redshift

## **INTRODUCTION**

Amazon Redshift is an enterprise wide secured, fully managed & scalable Data Warehouse solution that AWS provides

![image](https://user-images.githubusercontent.com/17497381/166972051-c4d404a4-1192-424c-aeb9-b4880ff9f5fc.png)

**AMAOZN REDSHIFT FEATUERS**
- Columnar Data storage
- MPP (Massive Parallel Processing)
- Concurrency scaling
- Materialized Views
- Automatic Caching
- Data Encryption: at Rest & in-transit

## **PRE-REQUESITES**
1. Please make sure that you have following pre-requisites full filled before starting with this hands-on session 
2. You need to have admin access to the AWS account to follow the steps mentioned below
3. Download the dataset from the Kaggle form [here](https://www.kaggle.com/competitions/nyc-taxi-trip-duration/data). For this you will have to sign-in into your Kaggle account.
4. Follow the section of Create IAM role as a part of Pre-requisites 
5. Follow the section of create an S3 bucket and uploading CSV datafiles which will be required while creating tables on Redshift

#### **CREATE IAM ROLE**

Before we go ahead and create a Redshift cluster lets first create an IAM role which we will need while creating a Redshift cluster. Please follow the setps mentioned below

1. Go to the IAM console in you AWS account. Click on Roles from the right hand side panel and then click on Create Role button

![image](https://user-images.githubusercontent.com/17497381/166972604-23fa1051-1b66-4766-837b-226a43474628.png)

2. In the _**Use cases for other AWS services**_ select Redshift from the drop down list. Then click on the _**Redshift - Customizable**_ radio button and then click on _**Next**_.

![image](https://user-images.githubusercontent.com/17497381/166972664-1baf4a4f-f5ef-4705-a1e9-d84df717adc8.png)

3. In the Add permission section, search for _**AmazonRedshiftAllCommandsFullAccess**_ policy. Its an AWS Managed policy so it will be a part of each AWS account. Select this policy and click on _**Next**_

![image](https://user-images.githubusercontent.com/17497381/166973961-9a2210a3-4df5-4b43-85a1-f14f5fefdd4f.png)

4. On the next page just provide _**Redshift-IAM-Role-1**_ as the name for the IAM role and then click on _**Create Role**_ button.

Congratulations! You have created your IAM role for the Redshift Cluster. Now lets proceed to create a Redshift cluster.


#### **CREATE S3 BCUKET TO SAVE CSV DATA FILES**

1. Open the S3 console on your AWS Account and then click on _**Create Bucket**_ button and fill in the following details for the same
  - Bucket Name - **aws-redshift-raw-csv**
    - _Note: You might not be able to give the same name to the S3 bucket which I have given, this is because the name of S3 bucket should be globally unique. So please give any name which works for you._
  - AWS Region - **US EAST N. Virginia (us-east-1)**
  - Keep all the other options default and just click on _**Create Bucket**_
 
 2. Once the S3 bucket is created, then click on the bucket and then  create a folder inside the bucket named as _**nyc-dataset**_ and click on the _**Create Folder**_ button.
![image](https://user-images.githubusercontent.com/17497381/166975802-6077062e-ce13-4c88-9b63-8f00badfd46f.png)

3. Once the folder is created then click on the folder and upload the _**train.csv**_ file obtained from the Kaggle link provided in the pre-requisites section Step 2 
  - **_NOTE: It might take a couple of minutes to upload the file, as the size of the file is ~193 MB._**

If you are following till this point then, Congratulations!! You have completed the required pre-requisites part for this hands-on session now we can start with Redshift part


### **CREATE A REDSHIFT CLUSTER**

1. Open the Redshift console on your AWS Account and click on _**Clusters**_ option on the left side panel and the click on _**Create Cluster**_
![image](https://user-images.githubusercontent.com/17497381/166976531-4f32b270-6a4f-4c85-93b7-fa536080a3b2.png)

2. Please fill in the following configurations details on this page
    - Cluster identifier - **redshift-cluster-1**
    - What are you planning to use this cluster for? - Select **Production**
    - Node Type - **dc2.large**
    - Number of Nodes - **1**
    - Admin user name - **test-user**
    - Admin user password - **Password1234**

3. In the Associated IAM roles section, click on _**Associated IAM role**_ button and then select the IAM role which created previously _**Redshift-IAM-Role-1**_ and then click on Associated IAM role button.
![image](https://user-images.githubusercontent.com/17497381/166977009-2516b6c6-6428-427e-93dd-6eb719ed26fa.png)

4. Keep the _**Additional Configurations**_ section default and do change anything over there and then just click on _**Create Cluster**_ button
![image](https://user-images.githubusercontent.com/17497381/166977083-3bb8325c-1a6f-4e8a-b435-7ba487dc05ff.png)

5. Cluster will take couple of minutes to get deployed. Have Patience for a while. Once the cluster is deployed, on the console you will see **Status** changed to **Available** (as shown below)
![image](https://user-images.githubusercontent.com/17497381/166977162-5c6a4871-0378-492c-a6f7-608a4132ecc6.png)


### **CREATE TABLE & LOAD DATA ON REDSHIFT CLUSTER

1. Before you start creating tables, lets just connect to the Redshift cluster using _**Query Editor V2**_. For that click on the Query Editor V2 in the Right hand side panel on the Redshift console.

2. This will open up a new tab and you will see the _**Redshift query editor v2**_. On this editor you will see the redshift cluster which we created in the right-hand side Database section.

3. Click on the redshift cluster name and it will automatically connect to you redshift cluster. If it does not connect automatically, then it will ask for user name and password for connection. Please enter the username and password which we used while creating the cluster
![image](https://user-images.githubusercontent.com/17497381/166977630-90158413-a9ad-4a92-b61c-f7e90a5c3a98.png)

4. Run the following query to Create the table _**taxi_rides**_
```
Create Table dev.public.taxi_rides (
  id VARCHAR(10),
  vendor_id INTEGER,
  pickup_datetime TIMESTAMP,
  dropoff_datetime TIMESTAMP,
  passenger_count INTEGER,
  pickup_longitude Decimal(9,6),
  pickup_latitude Decimal(9,6),
  dropoff_longitude Decimal(9,6),
  dropoff_latitude Decimal(9,6),
  store_and_fwd_flag VARCHAR(1),
  trip_duration INTEGER,
  primary key(id))
  DISTSTYLE KEY
  DISTKEY (id);
```

5. Run the following COPY command after successful creation of the table. Make sure that you update the following COPY command with you S3 Bucket name and with the AWS account ID as highlighted below
```
COPY dev.public.taxi_rides
from 's3://aws-redshift-raw-csv/nyc-dataset/train.csv' 
iam_role 'arn:aws:iam::284377223973:role/Redshift-IAM-Role-1'
FORMAT AS CSV
IGNOREHEADER 1 ;
```

6. After the COPY command is successfully executed, just the check the sample table records if they are properly inserted into the table using a Select * query
![image](https://user-images.githubusercontent.com/17497381/166978180-119961e7-7372-46d5-ab3a-f93cba6fec48.png)


### **LETS CREATE TABLE USING VISUAL EDITOR**

1. Before we create the table using the visual editor 2, lets just drop the table which is already created using the DROP command
```
DROP TABLE dev.public.taxi_rides;
```
2. Click on the **+ Create** button on the left side panel and then select _**Table**_ option from the list.
![image](https://user-images.githubusercontent.com/17497381/166978385-408e8912-dfe2-4a84-a7f6-fe620d6f4a03.png)

3. It will open up a modal, then mention the table name _**taxi_rides**_ in the text box provided. Then click on the _**Load from CSV**_ option to import the schema of the table directly rather than adding the columns names manually

4. Further just update the data types of the two columns _**pickup_datetime**_ and _**dropoff_datetime**_ to _**TIMESTAMP**_. Also click on the id column and then click on the radio button of _**Primary key**_.
  - _Note : Currently the UI option of creating table does not provide a way to add the precision for the decimal data type. So for now we will ignore it._
![image](https://user-images.githubusercontent.com/17497381/166978617-49687ee9-a53d-42c5-a423-074556c39179.png)

5. Further click on the _**Table Details**_ tab in the modal and then select the _**id**_ columns in the drop down of the _**distribution key**_. Similarly you can set the _**Sort Key**_ for the table accordingly. After this just click on the _**Create table**_ button.
![image](https://user-images.githubusercontent.com/17497381/166978740-4def029b-512d-408f-b49b-aea77c2e10b5.png)

6. Once the table is created successfully, then lets load the data into the table from the console itself. For this click on the _**Load Data**_ button on the left panel. Click on the _**Browse S3**_ to select the S3 location of the CSV files stored on the S3 bucket.

7. Then Select the IAM role from the drop down list, keep the format as CSV only, click on the _**data conversion parameter**_ and then scroll down to select the option of _**ignore header rows**_ and then select 1 in the given field.

8. In the target table just select the schema and the table name form the drop down as _**public**_ and _**taxi_rides**_ and then click on _**load data**_ button
![image](https://user-images.githubusercontent.com/17497381/166979051-820699b0-db0a-4ba8-81c8-223317550e46.png)

9. Once the the command successfully completes then just check the data inserted into the table using SELECT * query


### **CTAS - Create Table AS Query and UNLOAD command**

1. Lets create a table which provides an aggregated table of the KPIs by date and Hour
```
CREATE TABLE  dev.public.taxi_rides_date_hour AS
Select date(pickup_datetime) as trip_date,
extract(HOUR from pickup_datetime ) as trip_Hour,
count(distinct id) as Number_of_trips,
sum(passenger_count) as total_passenger,
ROUND(sum(trip_duration)/3600,2) as total_trip_duration_hour, 
max(trip_duration)/60 as max_trip_duration_minute
FROM 
dev.public.taxi_rides
Where  DATE(pickup_datetime)=DATE(dropoff_datetime)
Group by 1,2
Order by 1,2
```
2. Once you RUN  the above query it will give you the following table output
![image](https://user-images.githubusercontent.com/17497381/166979885-7b312a15-74f0-4354-ae1d-600efb1648b8.png)

3. As we have now created the table, lets now unload the content of the table back into S3 bucket which we have created using the UNLOAD command. Before running the UNLOAD query make sure that you update the S3 bucket name and the AWS Account ID in the command as highlighted below
```
UNLOAD ('SELECT * FROM dev.public.taxi_rides_date_hour')
to 's3://aws-redshift-raw-csv/rs-unload-output/' 
iam_role 'arn:aws:iam::284377223922:role/Redshift-IAM-Role-1'
HEADER
CSV;
```
4. Once the command executes, you can check your S3 bucket location, you will be able to see the files as shown below
![image](https://user-images.githubusercontent.com/17497381/166980031-df7a3264-e879-4d2d-97e1-c601d254b44e.png)


### **MATERIALIZED VIEWS**

1. Lets create an materialized view instead of the CTAS table that we created in the previous step. The Syntax is very similar to CTAS query.
```
CREATE MATERIALIZED VIEW dev.public.mv_date_hour_agg_trips as
Select date(pickup_datetime) as trip_date,
extract(HOUR from pickup_datetime ) as trip_Hour,
count(distinct id) as Number_of_trips,
sum(passenger_count) as total_passenger,
ROUND(sum(trip_duration)/3600,2) as total_trip_duration_hour, 
max(trip_duration)/60 as max_trip_duration_minute
FROM 
dev.public.taxi_rides
Where  DATE(pickup_datetime)=DATE(dropoff_datetime)
Group by 1,2
```
_NOTE : Here if you see that I am using the exact same query which we used in the CTAS query, the only difference that you can see is that I have removed the Order by clause from the end. This is because materialized views does not support the ORDER BY clause as it stores the table output in its memory for faster retrieval._

2. Lets query our Materialized view using SELECT * query.
![image](https://user-images.githubusercontent.com/17497381/166980294-95e6d36b-1014-4310-a6d7-65f652d75514.png)

3. Further let's say that the original table gets in the new incremental data and loads it into our _**taxi_rides**_ table, so our materialized view data becomes stale and inconsistent. So in this situation, you can refresh the materialized view using a simple query mentioned below or you can use a [**Automated refresh materialized view**](https://docs.aws.amazon.com/redshift/latest/dg/materialized-view-auto-mv.html)
```
REFRESH MATERIALIZED VIEW dev.public.mv_date_hour_agg_trips;
```

## CONCLUSION

Thus this blog covers the basics of Amazon Redshift and how do you work with Redshift starting from the creation of cluster to the tables and views. Its a very basic blog which will act as a boiler plate for the new users who are working with Amazon Redshift for the first time.

Once you are well versed with these queries and commands you can get into the depth of the configuration of each and ever command to the query used in this blog. I hope that this blog has helped you to see how easy it is to work with Amazon Redshift service. Happy Learning and Let me know if you have suggestions, feedbacks or questions in the comments section below.

## REFERENCES

#### 1. [Link to AWS Documentation - Query Editor V2](https://aws.amazon.com/blogs/aws/amazon-redshift-query-editor-v2-web-query-authoring/)
#### 2. [Link to AWS Documentation - COPY Command](https://docs.aws.amazon.com/redshift/latest/dg/r_COPY.html)
#### 3. [Link to AWS Documentation - UNLOAD Command](https://docs.aws.amazon.com/redshift/latest/dg/r_UNLOAD.html)
#### 4. [Link to AWS Documentation - Create Table AS Query (CTAS Query)](https://docs.aws.amazon.com/redshift/latest/dg/r_CREATE_TABLE_AS.html)
#### 5. [Link to AWS Documentation - Create Materialized Views](https://docs.aws.amazon.com/redshift/latest/dg/materialized-view-create-sql-command.html)
#### 6. [Link to AWS Documentation - Limitations and Refresh Views](https://docs.aws.amazon.com/redshift/latest/dg/materialized-view-refresh-sql-command.html)
#### 7. [Link to AWS Documentation - Auto Materialized Views](https://docs.aws.amazon.com/redshift/latest/dg/materialized-view-auto-mv.html)
