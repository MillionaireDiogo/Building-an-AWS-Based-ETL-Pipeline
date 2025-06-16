# Building an AWS-Based ETL Pipeline

## The Problem: A Retail Giant's Data Chaos

A large retail company, **ShopSphere**, had a growing digital customer base. Every day, millions of customers interacted with their website—browsing products, making purchases, abandoning carts, and signing up for newsletters. This activity generated a massive amount of raw data: customer profiles, transaction logs, browsing history, and preferences.

This data was automatically dumped into an **Amazon S3** bucket in raw **JSON** and **CSV** formats by various upstream applications.

However, the business teams were struggling. They had questions like:
- Who are our high-value customers?
- Which product categories have the most abandoned carts?
- How many repeat purchases do we see per region?

But they couldn't get timely answers.

The raw data was **unstructured and inconsistent**. Analysts had to manually wrangle files, write complex scripts, and spend hours just understanding the schema before any insights could be drawn.

It was clear: **ShopSphere needed a scalable, automated, and query-ready data pipeline**.

## The Solution: Building an AWS-Based ETL Pipeline

We designed a simple yet powerful workflow using four AWS services that work together seamlessly: **Amazon S3**, **AWS Glue Crawler**, **AWS Glue ETL Jobs**, and **Amazon Athena**—each playing a clear role in the journey from raw data to valuable insight.

### Step 1: Data Lands in Amazon S3

All customer activity—purchases, clicks, signups—is directed to a stream of raw data stored in **Amazon S3**, a secure and scalable storage service.

In this case, S3 serves as a **digital warehouse** where every piece of information is stored safely in folders and files, often in **JSON** and **CSV** formats.

---

### Step 2: AWS Glue Crawler

Next, we use **AWS Glue Crawler**, a service that explores the raw data in S3.

The crawler acts like a **smart librarian**:
- It scans the files,
- Determines the structure and types of information (e.g., customer names, product IDs, timestamps),
- And builds a **searchable catalog** (the **AWS Glue Data Catalog**).

This catalog allows other AWS tools to find, reference, and use the data easily.

---

### Step 3: AWS Glue ETL Job

**ETL** stands for **Extract, Transform, Load**—a process used to move and prepare data for analysis.

- **Extract** – Pull data from sources like apps, databases, or logs.
- **Transform** – Clean, reformat, and organize the data (e.g., fix errors, unify schemas, enrich content).
- **Load** – Store the cleaned data in its final destination, such as a refined S3 location.

With AWS Glue ETL jobs, we:
- Clean errors (e.g., missing or inconsistent values),
- Organize data by categories (e.g., region or date),
- And transform it into a structured, query-ready format.

The processed data is saved in a **separate S3 location**, clearly separated from the raw input—this is the **ready-to-use** version.

---

### Step 4: Query with Amazon Athena

Once the data is clean and organized, business teams can use **Amazon Athena** to query it.

Athena lets users write simple **SQL queries** to explore data directly in S3—no data movement or server setup required.

With Athena, teams can ask:
- "How many customers purchased last week?"
- "Which regions had the most abandoned carts?"
- "What products are gaining popularity?"

...and receive **fast, accurate answers**.

---

## The Outcome

This seamless ETL pipeline transformed a chaotic mess of raw data into a crystal-clear stream of insights:

- **Amazon S3** stored both raw and cleaned data.
- **AWS Glue Crawler** made sense of the stored data.
- **AWS Glue ETL jobs** cleaned, transformed, and restructured it.
- **Amazon Athena** unlocked the ability to ask business-critical questions and receive instant answers.

As a result, **ShopSphere** moved from **data overload** to **data intelligence**, enabling marketing, product, and executive teams to make smarter, faster decisions.

---

### Why This Approach Works

Using **Amazon S3, Glue, and Athena** provides a **scalable, cost-efficient, and serverless solution** for analyzing large volumes of raw or semi-structured data.

Unlike traditional databases:
- Storage and compute are separated.
- Data processing is flexible and on-demand.
- No infrastructure needs constant management.

This modern approach is ideal for analytics:
- Faster insights,
- Lower costs,
- Minimal maintenance.

## Prerequisites for the AWS ETL Workflow

Before setting up the ETL pipeline, ensure you have the following:

### ✅ An AWS Account

You'll need access to the **AWS Management Console** with sufficient permissions to create and manage the following services:
- Amazon S3
- AWS Glue
- Amazon Athena

---

### 🪣 Three S3 Buckets

You will need to create or specify three distinct S3 buckets:

- **Source Bucket**: Stores raw customer data (e.g., JSON, CSV).
- **Target Bucket**: Stores cleaned and transformed data after the ETL process.
- **Athena Results Bucket**: Stores query results from Amazon Athena.

---

### 🔐 IAM Role with Required Permissions

Create or assign an **IAM Role** that AWS Glue can assume.

This role must have:
- **Read access** to the Source S3 bucket.
- **Write access** to the Target S3 bucket.
- **Permissions for AWS Glue**, **Athena**, and the **Glue Data Catalog**.

---

### 📁 Data in the Source Bucket

Make sure the **source S3 bucket contains sample raw data files**, such as `.csv` or `.json`, which can be discovered and processed by the Glue crawler and ETL jobs.

---

### 🌍 Region Consistency

Ensure that **all AWS resources** (S3 buckets, Glue crawlers, Glue jobs, Athena queries) are created in the **same AWS region** to avoid integration issues.

## Getting Started

### Step 1: S3 Buckets Creation

Create the following S3 buckets with these exact names:

- **Source Bucket:** `source.bucketforetl.source`  
- **Target Bucket:** `target.bucketforetl.target`  
- **Athena Results Bucket:** `athenaresults.etl`
![Screenshot from 2025-06-16 09-46-54](https://github.com/user-attachments/assets/4fbe8d37-6b37-4d44-a558-b587f3d199c7)

### Step 2: IAM Roles and Permission Configurations

1. Navigate to the **IAM** service in the AWS Management Console.
2. Click on **Create role**.
3. Choose the service or use case as **Amazon Glue**.
4. Click **Next** to proceed with role creation.
![Screenshot from 2025-06-16 09-52-22](https://github.com/user-attachments/assets/f0c04408-5da6-42ca-a5d9-466b941df5cd)

### Step 3: Add Necessary Permissions to Amazon Glue

Attach the **AWSGlueServiceRole**, an IAM role that AWS Glue assumes when it runs your jobs, crawlers, and workflows. This role defines what Glue is allowed to access and perform on your behalf.

#### What the Role Needs to Do:
- Access S3 buckets (Source, Target, Athena Results)
- Read and write to the AWS Glue Data Catalog (e.g., create or update tables)
- Write logs and metrics via CloudWatch Logs
- Use Glue features (start jobs, etc.)

#### Creating the Role:
1. On the next page after choosing Amazon Glue service, **name the role** `AWSGlueServiceRoleETL`.
2. Click **Create role**.

> For this workflow, a custom permission policy will be created following the **principle of least privilege**, granting only the necessary read/write permissions to each S3 bucket. This policy will be attached **after** the role creation.

---

### Understanding What Access Is Needed

| Bucket Type           | Required Access                      |
|----------------------|------------------------------------|
| **Source Bucket**     | `s3:GetObject`, `s3:ListBucket`     |
| **Target Bucket**     | `s3:PutObject`, `s3:GetObject`, `s3:ListBucket` |
| **Athena Results Bucket** | `s3:PutObject`, `s3:GetBucketLocation`, `s3:ListBucket` |

---

### Attaching Custom Policy

1. Find the created role named `AWSGlueServiceRoleETL` in the **IAM console**.
2. Click **Add permissions** → **Attach policies** → **Create policy**.
3. Switch to the **JSON** tab.
4. Paste the JSON policy (provided below) into the editor.

*(JSON policy snippet to be provided next.)*
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "GlueReadSourceData",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::source.bucketforetl.source",
        "arn:aws:s3:::source.bucketforetl.source/*"
      ]
    },
    {
      "Sid": "GlueWriteTransformedData",
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::target.bucketforetl.target",
        "arn:aws:s3:::target.bucketforetl.target/*"
      ]
    },
    {
      "Sid": "AthenaWriteQueryResults",
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetBucketLocation",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::athenaresults.etl",
        "arn:aws:s3:::athenaresults.etl/*"
      ]
    }
  ]
}
```
![Screenshot from 2025-06-16 10-34-30](https://github.com/user-attachments/assets/dccc333f-6901-4bae-8857-511f48ebe149)
5. Click **Next**, then give the policy a name:  
   📛 **GlueLimitedS3AccessPolicy**
![Screenshot from 2025-06-16 10-35-44](https://github.com/user-attachments/assets/b21b044d-fe74-4161-be5b-727001532875)
![Screenshot from 2025-06-16 10-36-34](https://github.com/user-attachments/assets/137a1905-45bb-4894-a9f8-ef8c35ae6487)

### Step 3: Load Data into the Source Bucket in Amazon S3 (`source.bucketforetl.source`)

For this demo, use the provided `.csv` file containing fake customer data:  
🔗 [Download Demo Data (customer_data.csv)]

---

### ⚠️ Important Best Practice: Use a Folder, Not the Bucket Root

When working with CSV files for analytics using **AWS Glue** and **Amazon Athena**, avoid uploading data files directly to the **root** of an S3 bucket.

#### Why This Matters

If you upload a CSV file directly into the root of the bucket:
- AWS Glue Crawlers may correctly detect the metadata.
- However, **Athena queries might return empty results**, even though the table appears valid.

📖 Refer to the official AWS guidance on this known issue:  
[Why does my Amazon Athena query return empty results?](https://repost.aws/knowledge-center/athena-empty-results)

---

### ✅ Recommended Folder Structure

To ensure proper crawling and querying, follow this structure:

1. **Access the source S3 bucket:** `source.bucketforetl.source`
2. **Create a folder (prefix):**  
   - Click **Create folder**
   - Name the folder: `SalesData`
   - Click **Create folder**
3. **Upload the file:**  
   - Upload `customer_data.csv` into the newly created `SalesData` folder

Final S3 path should look like:  
`s3://source.bucketforetl.source/SalesData/customer_data.csv`




