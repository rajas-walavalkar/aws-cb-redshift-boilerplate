# aws-cb-redshift-boilerplate
This repository provides a good starting point for Amazon Redshift hands-on activity

**INTRODUCTION**
Amazon Redshift is an enterprise wide secured, fully managed & scalable Data Warehouse solution that AWS provides

![image](https://user-images.githubusercontent.com/17497381/166972051-c4d404a4-1192-424c-aeb9-b4880ff9f5fc.png)

**AMAOZN REDSHIFT FEATUERS**
- Columnar Data storage
- MPP (Massive Parallel Processing)
- Concurrency scaling
- Materialized Views
- Automatic Caching
- Data Encryption: at Rest & in-transit

**PRE-REQUESITES**
1. Please make sure that you have following pre-requisites full filled before starting with this hands-on session 
2. You need to have admin access to the AWS account to follow the steps mentioned below
3. Download the dataset from the Kaggle form here. For this you will have to sign-in into your Kaggle account.
4. Follow the section of Create IAM role as a part of Pre-requisites 
5. Follow the section of create an S3 bucket and uploading CSV datafiles which will be required while creating tables on Redshift

**CREATE IAM ROLE**
Before we go ahead and create a Redshift cluster lets first create an IAM role which we will need while creating a Redshift cluster. Please follow the setps mentioned below

1. Go to the IAM console in you AWS account. Click on Roles from the right hand side panel and then click on Create Role button

![image](https://user-images.githubusercontent.com/17497381/166972604-23fa1051-1b66-4766-837b-226a43474628.png)

2. In the _**Use cases for other AWS services**_ select Redshift from the drop down list. Then click on the _**Redshift - Customizable**_ radio button and then click on _**Next**_.

![image](https://user-images.githubusercontent.com/17497381/166972664-1baf4a4f-f5ef-4705-a1e9-d84df717adc8.png)

3. In the Add permission section, search for _**AmazonRedshiftAllCommandsFullAccess**_ policy. Its an AWS Managed policy so it will be a part of each AWS account. Select this policy and click on _**Next**_

![image](https://user-images.githubusercontent.com/17497381/166973961-9a2210a3-4df5-4b43-85a1-f14f5fefdd4f.png)

4. On the next page just provide _**Redshift-IAM-Role-1**_ as the name for the IAM role and then click on _**Create Role**_ button.

Congratulations! You have created your IAM role for the Redshift Cluster. Now lets proceed to create a Redshift cluster.

**CREATE S3 BCUKET TO SAVE CSV DATA FILES**
