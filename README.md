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
🔗 [Download Demo Data (customer_data.csv)](https://docs.google.com/document/d/1wuldNeTHByOohjwFtPWU-AWzzH3f1wfiTfxXU3GBoLU/edit?tab=t.0) (download as docx. or pdf. anduse a doc converter tool to change from doc to csv)

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
![Screenshot from 2025-06-16 11-21-26](https://github.com/user-attachments/assets/a49cff00-3a54-4e14-8dbe-05b4c78f3f1c)
Final S3 path should look like:  
`s3://source.bucketforetl.source/SalesData/customer_data.csv`

This structure helps Athena properly read partitioned data and aligns with best practices for building a reliable data lake.

### Step 4: Transform Data with Amazon Glue

#### 🔍 What is a Glue Crawler?

An **AWS Glue Crawler** is a tool that:
- Scans data stored in Amazon S3
- Detects its structure (columns, data types, file format)
- Automatically creates or updates a table in the **Glue Data Catalog**

This table acts like a **schema map**, allowing services like **Amazon Athena** to understand how to read and query the dataset.

---

### 🧪 Our Use Case

In this project, the crawler will scan the `customer_data.csv` file located inside the `SalesData` folder in the source S3 bucket:

It will then create a corresponding table in the **AWS Glue Data Catalog**, making it ready for ETL jobs or SQL querying via Athena.

---

### 🛠️ Steps to Create the Crawler

1. Go to **AWS Glue** → **Data Catalog** → **Crawlers**
2. Click **Create Crawler**
3. **Name your crawler:** `gluecrawler`  
   Click **Next**

4. You’ll be asked:  
   ❓ *Is your data already mapped to Glue tables?*  
   Choose: **Not yet**

5. **Add a Data Source:**
   - Choose **S3** as the data source
   - Click **Browse** and navigate to:
     ```
     s3://source.bucketforetl.source/SalesData/
     ```
![Screenshot from 2025-06-16 11-51-08](https://github.com/user-attachments/assets/dfc8f23b-18b0-4718-a48d-5846ac66f8b6)

Continue through the wizard (we can add those steps next if you'd like), and once the crawler runs successfully, a table will appear in the **Glue Data Catalog**, describing the dataset.

6. Select the final option: **Crawl all sub-folders**
7. Click **Add an S3 data source**, then click **Next**

---

### 🔐 Security Settings

8. On the **Security settings** page:
   - For **IAM Role**, select the role you created in **Step 2**:
     ```
     AWSGlueServiceRoleETL
     ```
   - Click **Next**
![Screenshot from 2025-06-16 11-55-55](https://github.com/user-attachments/assets/6dc074f7-4dfc-447a-978d-423b44112f01)

---

### 🗃️ Output Configuration

9. On the **Output configuration** page:
   - For **Target database**, click on **Add database**
   - Create a new database with the name:
     ```
     saledata
     ```
   - Click **Create**, then select `saledata` as the target database
   - Click **Next**
![Screenshot from 2025-06-16 12-12-15](https://github.com/user-attachments/assets/61e2c9d3-6b8a-4e67-9c70-961dd7e35178)

---

### 🗃️ Output Configuration (continued)

10. Back on the **Output configuration** page:
   - For **Target database**, select the newly created database:
     ```
     saledata
     ```
   - Click **Next**

---

### 🕒 Crawler Schedule

11. On the **Crawler schedule** page:
   - Click the dropdown and select: **Run on demand**

> Choosing the “On Demand” schedule means the crawler will only run when manually triggered.  
This is ideal for development or experimental pipelines like this one, where automatic, recurring crawls (hourly, daily, etc.) are not yet necessary.

![Screenshot from 2025-06-16 12-15-42](https://github.com/user-attachments/assets/8096ca1c-6392-40b6-b362-939f28552750)

---

### ✅ Create and Run the Crawler

12. **Create the Crawler** by reviewing your settings and clicking **Finish**

13. Now that the crawler has been successfully created, it's time to **run it**:

- Navigate to the list of crawlers
- Select the crawler named `gluecrawler`
- Click **Run Crawler**

> Whenever this crawler is executed, it connects to the `customer_data.csv` file in your source S3 bucket (inside the `SalesData` folder). It scans the file, detects its structure (columns, data types, file format), and creates or updates a table in the **AWS Glue Data Catalog**.

This generated table serves as a schema reference for services like **Amazon Athena**, enabling them to correctly read and query the data.

Once the crawler completes:
- Go to **Databases** under **AWS Glue Data Catalog**
- Open the `saledata` database
- You’ll see the generated table, ready for querying or transformation
![Screenshot from 2025-06-16 12-26-13](https://github.com/user-attachments/assets/0931a8e3-a412-4435-842b-d665d572d996)

![Screenshot from 2025-06-16 12-28-12](https://github.com/user-attachments/assets/a75977dd-f303-4be3-92e4-28b3d0d74360)

---

### 🔍 What Happens to the Database After the Crawler Runs?

Before running the crawler, your Glue database was essentially an empty container—a placeholder with no tables or metadata.

Once the crawler completes, it:

- ✅ Scans the file (`customer_data.csv`) in your S3 source location.
- ✅ Detects the schema — column names, data types, format (CSV), etc.
- ✅ Creates a table inside the previously empty database you selected during setup.

The database now contains a structured table representing your data, stored in the **Glue Data Catalog**.

This table is what services like **AWS Glue ETL jobs** and **Amazon Athena** use to reference the data for transformations or queries. You don’t need to define the schema manually—Glue handles this automatically.

---

### 🔎 How to Access the Database and Table

Navigate to:
AWS Glue → Data Catalog → Databases → saledata → Tables
Here you will find an overview of the tables created by the crawler.
![Screenshot from 2025-06-16 12-31-26](https://github.com/user-attachments/assets/4d723023-b066-424e-8fdc-086af03266db)

## Step 5: Creating and Running an ETL Job in AWS Glue

### 1. Create the Job
- Navigate to **AWS Glue Console** → **ETL Jobs**
- Click **Create job**
- Choose **Visual with a source and target (Visual ETL)**

---

### 2. Set the Source
- Under **Source**, choose **Amazon S3**
- Click the source node to open the **Data Source Properties – S3** panel
![Screenshot from 2025-06-16 12-59-32](https://github.com/user-attachments/assets/00fd5164-1272-49a1-b44d-872176edd934)
![Screenshot from 2025-06-16 12-41-02](https://github.com/user-attachments/assets/520a25be-bf4d-4b98-b9ba-b781b1fb3f49)

> **What Does This Panel Do?**  
> It lets you specify what data Glue should read, using the table created by your crawler.

- Click **Data catalog table**
- Select the database you created earlier (e.g., `saledata`)
- Select the table (e.g., `salesdata`)
- Choose your IAM role (the one with the necessary S3 and Glue access)
- Click **Save**

✅ At this point, your data source is fully configured.

---

### 3. Set the Target
> **What Is the Target Location?**  
> This is where the transformed data will be saved — in your target S3 bucket (created earlier), in a new format (e.g., JSON). This becomes the “clean” version of your data, ready for querying.

- Click the plus (+) icon next to the source node
- Select **Add Target** → choose **Amazon S3**
- Choose **JSON** as the output format (or **Parquet** if preferred)
- Select the target S3 bucket created in Step 1 (e.g., `target.bucketforetl.target`)
![Screenshot from 2025-06-16 13-02-11](https://github.com/user-attachments/assets/a134b2d6-e372-4198-baf8-628e697e85ae)

---

### 4. Add a Transformation (Optional but Recommended)

> **What Action Is Performed Here?**  
> A transformation lets you modify the data—such as selecting specific columns, filtering rows, or changing data types—before it’s saved to the target.

- Click the plus (+) icon again and select **Transform**
- Choose **Select Fields**
- Under **Node Parent**, choose the source node (Amazon S3)

This transformation node lets you pick which fields from the original dataset you want to include in the final output.
![Screenshot from 2025-06-16 13-18-00](https://github.com/user-attachments/assets/2ebcc4f7-362f-45ff-a8e5-f0385d922ac8)

---

### 5. Set the Target Node’s Parent

- Click on the **target node**
- Change its **Parent** to be the **Select Fields** node (instead of the source)

This ensures the target receives the transformed data, not just a raw copy.
![Screenshot from 2025-06-16 13-22-03](https://github.com/user-attachments/assets/201fd9b3-f7e9-4fff-9b64-568c056f6b45)

---

### 6. Finalize and Run the Job

- Go to the **Job Details** section
- Ensure your IAM role is selected (it must have read/write access to both S3 buckets and AWS Glue)
- Give the job a name if you haven’t already
- Click **Save**, then click **Run Job**

---

### Result

AWS Glue will:

- Read the CSV file from your **source S3 bucket**
- Optionally transform the data (e.g., filter fields)
- Write the output in **JSON format** into the **target S3 bucket**

You can monitor the job by clicking on the run details to confirm that the job has completed successfully.
![Screenshot from 2025-06-16 13-37-44](https://github.com/user-attachments/assets/7414cae0-ba66-4d8f-8911-f1e9c0b36b35)

---

## After the ETL Job Runs Successfully

Once the ETL job completes, navigate to your **target S3 bucket**.

There, you will find a newly created object (file) containing the transformed data in the format you selected (e.g., JSON).

The ETL job has effectively:
- Extracted the raw data from the source bucket
- Transformed the data (e.g., selecting specific fields or changing formats)
- Loaded the clean data into the target bucket
![Screenshot from 2025-06-16 13-43-00](https://github.com/user-attachments/assets/44182f73-1e48-4cab-ba00-1eeb5940574b)

---

> The ETL job automates the process of preparing raw data for analytics,  
> making it organized, structured, and ready for querying with services like **Amazon Athena**.

---

## Cleanup Steps for Your AWS Glue ETL Project

1. **Delete S3 Data (Optional if Not Needed)**
   - Go to the **source** and **target** S3 buckets.
   - Delete any no longer needed files:
     - Uploaded raw files (e.g., `customer_data.csv`)
     - Transformed output files (e.g., JSON or Parquet)
     - Athena query results stored in the Athena results bucket

2. **Delete the Glue Job**
   - Navigate to **AWS Glue Console** → **ETL Jobs**
   - Select the ETL job you created
   - Click **Actions** → **Delete**

3. **Delete the Glue Table and Database**
   - Go to **AWS Glue Console** → **Data Catalog** → **Tables**
   - Delete the table created by the crawler (e.g., `salesdata`)
   - Then go to **Databases**, and delete the database (e.g., `saledata`) if no longer needed

4. **Delete the Glue Crawler**
   - Go to **AWS Glue Console** → **Crawlers**
   - Select and delete the crawler you created

5. **Detach or Delete IAM Role (Optional)**
   - If the IAM role was created specifically for this project:
     - Go to **IAM Console** → **Roles**
     - Select the Glue role
     - Detach policies or delete the role if it’s no longer needed

6. **Review and Delete Athena Query History (Optional)**
   - Go to **Amazon Athena Console**
   - Optionally clear saved queries or delete query results from the Athena results S3 bucket

---

### Project Inspiration

This project is inspired by:  
[YouTube - AWS Glue ETL Pipeline Project](https://www.youtube.com/watch?v=1IJJmWKzeXQ&list=PLneBjIzDLECkz7SwM-HFqZsvAZDz4hcpq&index=3)




