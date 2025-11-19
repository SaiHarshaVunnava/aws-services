
# Serverless Spark ETL Pipeline on AWS

This project implements a fully automated, serverless, event-driven Spark ETL pipeline using AWS Lambda and AWS Glue.
When a raw CSV file is uploaded into S3, the pipeline automatically triggers a Glue Spark job, processes the data, runs analytical queries, and stores the results back into S3 ― without any manual execution.

---

## 📊 Project Overview

This pipeline replaces manual ETL with automation:

1. Upload reviews.csv → S3 Landing Bucket

2. S3 triggers → AWS Lambda

3. Lambda starts the AWS Glue Spark ETL job

4. Glue processes file using PySpark + Spark SQL

5. Cleaned + aggregated results saved → S3 Processed Bucket

The result is a real-world serverless data engineering pipeline.

---

## 🏗️ Architecture



**Data Flow:**
`S3 (Upload) -> Lambda (Trigger) -> AWS Glue (Spark Job) -> S3 (Processed Results)`

---

# 🛠️ **Technologies Used**

| Component | AWS Service |
|----------|-------------|
| Storage | Amazon S3 |
| Serverless Trigger | AWS Lambda |
| ETL Engine | AWS Glue (Spark) |
| Processing | PySpark + Spark SQL |
| Security | AWS IAM |

---

## 🔧 Setup and Deployment

Follow these steps to deploy the pipeline in your own AWS account.
## **1. Create S3 Buckets**

### **Landing Bucket**
handsonfinallanding
### **Processed Bucket**
handsonfinalprocessed


---

## **2. Create IAM Role for Glue**

1. Go to **IAM → Roles → Create Role**
2. Select **AWS Service → Glue**
3. Attach:
   - `AWSGlueServiceRole`
   - `AmazonS3FullAccess`
4. Name it: AWSGlueServiceRole-Reviews

---

## **3. Create the AWS Glue Spark ETL Job**

1. Open **AWS Glue → ETL Jobs → Create Job**
2. Choose **Spark Script Editor**
3. Paste the code from: src/glue_job_script.py
4. Job Details:
   - **Name:** `process_reviews_job`
   - **Role:** `AWSGlueServiceRole-Reviews`
5. Save.

---
## **4. Create the Lambda Trigger Function**

1. Go to **AWS Lambda → Create Function**
2. Choose **Author from Scratch**
3. Set:
   - Function Name: `start_glue_job_trigger`
   - Runtime: Python 3.10
4. Create function.

Paste code from: src/lambda_function.py

Make sure:
```python
GLUE_JOB_NAME = "process_reviews_job"
```

Click Deploy.


### 5. Create the Lambda Trigger Function
This function will start the Glue job when a file is uploaded.

1.  Go to the **AWS Lambda** service and **Create function**.
2.  Select **Author from scratch**.
3.  Set the **Function name** to `start_glue_job_trigger`.
4.  Set the **Runtime** to **Python 3.10** (or any modern Python runtime).
5.  **Permissions:** Under "Change default execution role," select **Create a new role with basic Lambda permissions**. This role will be automatically named.
6.  Create the function.

#### 5a. Add Lambda Code
Paste the contents of `src/lambda_function.py` into the code editor. Make sure the `GLUE_JOB_NAME` variable matches the name of your Glue job (`process_reviews_job`).

#### 5b. Add Lambda Permissions
The new Lambda role needs permission to start a Glue job.
1.  Go to the function's **Configuration** > **Permissions** tab and click the role name.
2.  In the IAM console, click **Add permissions** > **Create inline policy**.
3.  Use the JSON editor and paste the following policy:
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": "glue:StartJobRun",
                "Resource": "*"
            }
        ]
    }
    ```
4.  Name the policy `Allow-Glue-StartJobRun` and save it.

#### 5c. Add the S3 Trigger
1.  Go back to your Lambda function's main page.
2.  Click **Add trigger**.
3.  Select **S3** as the source.
4.  Select your `handsonfinallanding` bucket.
5.  Set the **Event type** to `s3:ObjectCreated:*` (or "All object create events").
6.  Acknowledge the recursive invocation warning and click **Add**.

---

## **6. Test the Pipeline**

Upload the file: reviews.csv
To this S3 path: s3://handsonfinallanding/

### **What happens automatically:**
- Lambda is triggered  
- Lambda starts the Glue ETL job  
- Glue reads, transforms, and analyzes the data  
- Processed output is saved to the processed S3 bucket  

---

## 📈 **7. Output Data in Processed S3 Bucket**

The Glue ETL job writes results to the following paths:
s3://handsonfinalprocessed/processed-data/
s3://handsonfinalprocessed/Athena Results/daily_review_counts/
s3://handsonfinalprocessed/Athena Results/top_5_customers/
s3://handsonfinalprocessed/Athena Results/rating_distribution/

All outputs are stored as **Parquet files**, optimized for analytics.

---


## 📈 Query Results

After the job (which may take 2-3 minutes to run), navigate to your `handsonfinalprocessed` bucket. You will find the results in the `Athena Results/` folder, organized into sub-folders for each query:

* `s3://handsonfinalprocessed/Athena Results/daily_review_counts/`
* `s3://handsonfinalprocessed/Athena Results/top_5_customers/`
* `s3://handsonfinalprocessed/Athena Results/rating_distribution/`

You will also find the complete, cleaned dataset in `s3://handsonfinalprocessed/processed-data/`.

---
## 🧹 Cleanup

To avoid any future charges (especially if you're on the Free Tier), be sure to delete the resources you created:
1.  Empty and delete the `handsonfinallanding` and `handsonfinalprocessed` S3 buckets.
2.  Delete the `start_glue_job_trigger` Lambda function.
3.  Delete the `process_reviews_job` Glue job.
4.  Delete the `AWSGlueServiceRole-Reviews` IAM role.

### **1. S3 Buckets Overview**
- Screenshot showing both buckets:
<img width="739" height="389" alt="image" src="https://github.com/user-attachments/assets/1fa7aa07-8fd6-48fb-b603-6c239dec528b" />

### **2. IAM Role for Glue**
- Role: `AWSGlueServiceRole-Reviews`
- Attached policies:
  - AWSGlueServiceRole  
  - AmazonS3FullAccess
<img width="589" height="435" alt="image" src="https://github.com/user-attachments/assets/94f21e0a-e9d7-4558-9581-4f80db7a3d4e" />

### **3. Glue Job – Job Configuration Page**
- Job name: `process_reviews_job`
- IAM role
- Spark job settings
<img width="670" height="382" alt="image" src="https://github.com/user-attachments/assets/ebec692e-c77f-468e-bb32-518547b6fc4d" />

### **4. Glue Job – Script Editor**
- The full ETL PySpark script  
- Shows input/output S3 paths  
- Shows your additional 3 SQL queries (required)
<img width="665" height="542" alt="image" src="https://github.com/user-attachments/assets/247ec9af-859c-404d-826f-eb88b5c4e86f" />

### **5. Lambda Function – Overview Page**
- Function name: `start_glue_job_trigger`
- Runtime: Python 3.10
- Basic configuration
<img width="456" height="364" alt="image" src="https://github.com/user-attachments/assets/c4064cd3-13d8-48ed-adf4-fad84495c836" />

### **6. Lambda Function – Code Editor**
- Paste of `lambda_function.py`
- Visible `GLUE_JOB_NAME = "process_reviews_job"`
<img width="631" height="524" alt="image" src="https://github.com/user-attachments/assets/eacb7d17-6118-4840-bdca-de1393eaf648" />

### **7. Lambda S3 Trigger**
- Trigger type: S3
- Bucket: `handsonfinallanding`
- Event type: `s3:ObjectCreated:*`
<img width="426" height="346" alt="image" src="https://github.com/user-attachments/assets/8924da12-c489-4bb6-b091-f21431505ee3" />

### **8. Uploading the Input File**
- Screenshot showing `reviews.csv`
<img width="659" height="543" alt="image" src="https://github.com/user-attachments/assets/3004a2d2-63af-45a9-85c7-646c1728a762" />

### **9. Processed Output – Folder Structure**
<img width="654" height="541" alt="image" src="https://github.com/user-attachments/assets/812aae1b-c65c-4499-94d6-ffc1b51c0363" />
<img width="817" height="540" alt="image" src="https://github.com/user-attachments/assets/245338a6-176e-4810-9941-33b2298ea848" />

















