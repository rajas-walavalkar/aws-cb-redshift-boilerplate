# aws-cb-redshift-boilerplate : Session 2
This folder is a continuation of Session 1 which you can find in the **CB-Session-1** folder in this same repository. It provides a complete end to end hands-on activity which you can try out on personal AWS account. Follow through the steps mentioned below carefully to become master of Aamzon Redshift

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

Please make sure that you have following pre-requisites full filled before starting with this hands-on session 
1. Please go through the Session 1 Lab manual and then start refering to this lab manual as couple of foundational things are already taken care in the presvious sessions Lab manual
2. You need to have admin access to the AWS account to follow the steps mentioned below
3. Download the dataset from the github repo from the **CB-Session-2** in the dataset folder and push it to an S3 bucket
4. Follow the section of Create IAM role as a part of Pre-requisites 
5. Follow the section of create an S3 bucket and uploading CSV datafiles which will be required while creating tables on Redshift

#### **CREATE A GLUE CRAWLER FOR THE S3 DATASET**

1. Go to the IAM console in you AWS account. Search for Glue service which will open up the Glue console. The click on the Crawler link in the left hand side pannel

![image](https://user-images.githubusercontent.com/17497381/172198319-388e026b-2e6a-41e5-af99-654b2907a9a7.png)

2. Then click on _**Add Crawler**_. After that provide the following information to configure the Glue Crawler
- Crawler Name : **crawl-s3-latlong-metadata**
- Choose a data store : **S3**
- In Include s3 path select the path where the **latlong-mapping.csv** is uploaded
- IAM Role : specify the role name of the new IAM role AWSGlueS3Role
- Frequency : Select **Run on Demand**
- Click on Add Database : **rs-lat-long-database**

![image](https://user-images.githubusercontent.com/17497381/172199564-5a97c7f2-cb0e-44da-a356-35e7d2857ee7.png)
![image](https://user-images.githubusercontent.com/17497381/172199679-389f182b-c963-493a-b5f1-7f0795062e4c.png)
![image](https://user-images.githubusercontent.com/17497381/172199843-4f387300-043f-4979-b6db-f68f7d75c000.png)

3. Once the Crawler is created, then select the crawler and click on **Run Crawler**

4. The crawler will execute and complete within few minutes. Once its completed, you will see the number of tables added as 1 as shown in the figure below

![image](https://user-images.githubusercontent.com/17497381/172200496-56c3a3ae-ea9b-4914-adff-6558ea7d23c5.png)

5. You can check the table which is created in the Glue Catalog after the crawler is successfull

![image](https://user-images.githubusercontent.com/17497381/172200689-4084ab3f-6004-47e7-80f1-1342ae4271ef.png)

Congratulations! You have populated the **Glue Data Catalog** with the external table stored on S3 bucket 


#### **CREATE EXTERNAL TABLE (SPECTRUM TABLE) ON REDSHIFT CLUSTER**

1. Before we create the Redshift external table, we will have to update the IAM role which we created in our previous Session by adding two more IAM policies. Goto the IAM console and find the IAM role named as __**Redshift-IAM-Role-1**__ and then click on __**Add Permission**__, Search for two IAM policies __**AWSGlueConsoleFullAccess**__ & __**AmazonS3ReadOnlyAccess**__ and then click on **Attach Policies**. These IAM roles will provide the Redshift cluster to access the AWS GLue Catalog and the S3 bucket.

2. Open the Redshift console on your AWS Account and then start the Redshift cluster which you had created in the CB-Session-1. Once the cluster is in running state, then open the visual editor 2 from the console and then connect to the Redshift cluster.

![image](https://user-images.githubusercontent.com/17497381/172207259-8a641f3f-08e2-4c82-b524-b6512232963c.png)
 
3. Once you are connected to your Redshift cluster, then run the following command in the query editor to create an external schema which is ppinting towards AWS Glue Catalog.

```
CREATE EXTERNAL SCHEMA rs_glue_schema FROM DATA CATALOG 
DATABASE 'rs-lat-long-database' 
iam_role 'arn:aws:iam::284377223972:role/Redshift-IAM-Role-1' 
region 'us-east-1';
```

4. Once you run the query which created the external schema, just the refresh the Schema pannel and then you will be able to see the table which we created in our previous step using a Glue Crawler

![image](https://user-images.githubusercontent.com/17497381/172211594-840b3ff7-a9ff-413b-82e0-c613c4c6f3ab.png)

5. Lets check whether we are able to query the data which is present in the S3 bucket from the Redshift cluster using the Glue external Catalog. Once you run the following query you should see the following table

```
SELECT * FROM rs_glue_schema.nyc_latlong_mapping;
```

![image](https://user-images.githubusercontent.com/17497381/172212341-1c45393f-94fb-4e61-b276-2e2f82e3c2fa.png)


If you are following till this point then, Congratulations! You have successfully created a Redshift Spectrum Table using External **Glue Data Catalog** for the data stored on S3 bucket 


### **CREATE A MATERIALIZED VIEW TO JOIN LOCAL TABLE WITH SPECTRUM TABLE**

1. Now we have two tables with us, one which is a local table which we had loaded using COPY command in our previous session and one which we created in this session as a spectrum table
Local Table - **dev.public.taxi_rides**
Spectrum Table - **rs_glue_schema.nyc_latlong_mapping**

![image](https://user-images.githubusercontent.com/17497381/172410145-41a15416-1074-4e10-9135-e577b43c4cea.png)

The **taxi_rides** table have latitude and longitude information for all the rides in NYC and the **nyc_latlong_mapping** have all the zipcode mappings for all the relavant latitude and longitudes present in NYC. So lets create a materialized views which provides the zipcode information for every ride by joining both the above tables

2. Execute the below query which creates materialized view and joins both the tables
3. 
```
CREATE MATERIALIZED VIEW dev.public.trip_nyc_wtih_zip as
SELECT 
id,vendor_id,pickup_datetime,dropoff_datetime,
passenger_count,pickup_longitude,pickup_latitude,
dropoff_longitude,dropoff_latitude,
store_and_fwd_flag,trip_duration, 
'New York' as state,
ltp. zipcode as pickup_zipcode,
ltd. zipcode as dropoff_zipcode
FROM dev.public.taxi_rides tr 
INNER JOIN 
rs_glue_schema.nyc_latlong_mapping ltp
ON tr. pickup_latitude = ltp. latitude
AND tr. pickup_longitude = ltp. longitude
INNER JOIN 
rs_glue_schema.nyc_latlong_mapping ltd
ON tr. dropoff_latitude = ltd. latitude
AND tr. dropoff_longitude = ltd. longitude;
```

And then run Select * query on the materialized view created to get the joined result

![image](https://user-images.githubusercontent.com/17497381/172412004-40faf51a-d721-4deb-9d86-1bdbbb7a9c58.png)

We have completed our Spectrum tutorial if you are following till here.

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
