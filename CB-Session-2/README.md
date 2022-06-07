# aws-cb-redshift-boilerplate : Session 2
This folder is a continuation of Session 1 which you can find in the **CB-Session-1** folder in this same repository. It provides a complete end to end hands-on activity which you can try out on personal AWS account. Follow through the steps mentioned below carefully to become master of Aamzon Redshift

## **INTRODUCTION**

Amazon Redshift is an enterprise wide secured, fully managed & scalable Data Warehouse solution that AWS provides

![image](https://user-images.githubusercontent.com/17497381/166972051-c4d404a4-1192-424c-aeb9-b4880ff9f5fc.png)

**AMAOZN REDSHIFT FEATUERS**
- Concurrency scaling
- Materialized Views
- Redshift Spectrum
- External Schemas
- Workload Management (WLM)

## **PRE-REQUESITES**

Please make sure that you have following pre-requisites full filled before starting with this hands-on session 
1. Please go through the Session 1 Lab manual and then start refering to this lab manual as couple of foundational things are already taken care in the presvious sessions Lab manual
2. You need to have admin access to the AWS account to follow the steps mentioned below

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

### **WORKLOAD MANAGEMENT CONFIGURATION ON REDSHIFT**

1. Open up the Redshift console and then click on **Workload management** section in the left hand side panel. Once the Workload management page opens up, you would see a default parameter group which will be already there. Lets create a new parameter group as we cannot update the default parameter group. 

![image](https://user-images.githubusercontent.com/17497381/172433478-9f94fc3a-6758-4d6e-96f7-24bb9b966d28.png)


2. Click on the **Create** button and then enter the name of the parameter group and description for the same
Name - test-parameter-group-wlm
Decrpiption - test
By default the parameter groupyou created has a default queue which is an Auto WLM. Lets understand the parameters by clicking on Edit workload group button.

3. When you are in the edit option, you can add more queues to the parameter group. Now as we have Auto WLM selected we cannot update the memory config and the concurrency slot for the queue, you can only set the priority and the add query monitroing rules as required.

4. If you switch to manual WLM mode for the parameter group, then all the queues are to be configured manually by adding memory and concurrency slots per queues along with the priority and query monitoring options.

5. Using Query monitoring options you can change the priority of the queires, trigger an alert or abort the queries. You can define the action as required if the montoring condition is staisfied.

## CONCLUSION

Thus this blog is a continuation of the previous session, and it gives in depth understanding of the Redshift features like External Scehma, Redshift Spectrum * Workload Management

Once you are well versed with these queries and commands you can get into the depth of the configuration of each and ever command to the query used in this blog. I hope that this blog has helped you to see how easy it is to work with Amazon Redshift service. Happy Learning and Let me know if you have suggestions, feedbacks or questions in the comments section below.

## REFERENCES

#### 1. [Link to AWS Documentation - Redshift Spectrum](https://docs.aws.amazon.com/redshift/latest/dg/c-using-spectrum.html#c-spectrum-overview)
#### 2. [Link to AWS Documentation - Setting up Glue Crawlers](https://docs.aws.amazon.com/glue/latest/dg/crawler-configuration.html)
#### 3. [Link to AWS Documentation - Glue Catalog](https://docs.aws.amazon.com/glue/latest/dg/populate-data-catalog.html)
#### 4. [Link to AWS Documentation - External Schema](https://docs.aws.amazon.com/redshift/latest/dg/r_CREATE_EXTERNAL_SCHEMA.html)
#### 5. [Link to AWS Documentation - Create Materialized Views](https://docs.aws.amazon.com/redshift/latest/dg/materialized-view-create-sql-command.html)
#### 6. [Link to AWS Documentation - Workload Management (WLM)](https://docs.aws.amazon.com/redshift/latest/dg/c_workload_mngmt_classification.html)
#### 7. [Link to AWS Documentation - Auto WLM & Manual WLM](https://docs.aws.amazon.com/redshift/latest/dg/automatic-wlm.html)
