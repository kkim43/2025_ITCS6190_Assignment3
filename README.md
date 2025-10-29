# ITCS-6190 Assignment 3: AWS Data Processing Pipeline

**Name(ID):** Kiyoung Kim (801426261)  
**E-Mail:** kkim43@charlotte.edu

This project demonstrates an end-to-end serverless data processing pipeline on AWS. The pipeline automates the flow of data from raw ingestion to visualization without manual intervention. Raw order data is uploaded to Amazon S3, automatically processed by an AWS Lambda function, cataloged using AWS Glue, queried through Amazon Athena, and finally visualized on a Flask-based web dashboard hosted on an EC2 instance.

## 1. Amazon S3 Bucket Structure 🪣

**Approach:**  
I created an S3 bucket to organize data into three main folders.  
Each folder has a clear purpose — the Lambda function reads from `raw/`, saves filtered files to `processed/`, and Athena saves query results to `enriched/`.

**Explanation:**  
The S3 bucket is named `itcs6190.assignment3.801426261.kkim43` and has the following structure:

```
├── raw/        # Raw data files uploaded here  
├── processed/  # Cleaned and filtered data from Lambda  
└── enriched/   # Athena query results stored here  
```

This setup keeps files tidy and makes it easy for Lambda, Glue, and Athena to work together.

**Screenshot:**  
📸 *Figure 1.* Amazon S3 Bucket Structure  
![Screenshot1.png](./Screenshot/Screenshot1.png)

---
## 2. IAM Roles and Permissions 🔐

**Approach:**  
To let AWS services work together safely, I created three IAM roles — one for Lambda, one for Glue, and one for EC2.  
Each role has the right permissions so that every service can only do what it needs.

**Explanation:**  
- **Lambda-S3-Processing-Role-Assignment3**  
  - Used by: Lambda  
  - Policies: `AWSLambdaBasicExecutionRole`, `AmazonS3FullAccess`  

- **Glue-S3-Crawler-Role-Assignment3**  
  - Used by: AWS Glue  
  - Policies: `AmazonS3FullAccess`, `AWSGlueConsoleFullAccess`, `AWSGlueServiceRole`  

- **EC2-Athena-Dashboard-Role-Assignment3**  
  - Used by: EC2 instance  
  - Policies: `AmazonS3FullAccess`, `AmazonAthenaFullAccess`  

These roles make sure that Lambda can access S3, Glue can crawl the processed data, and EC2 can query Athena securely.

**Screenshot:**  
📸 *Figure 2.* IAM Roles Created  
![Screenshot2.png](./Screenshot/Screenshot2.png)

---

## 3. Create the Lambda Function ⚙️

**Approach:**  
I created a Lambda function that automatically processes new files uploaded to the `raw/` folder in S3.  
This function reads the raw CSV file, filters it, and saves the cleaned version to the `processed/` folder.

**Explanation:**  
- **Function name:** `FilterAndProcessOrders`  
- **Runtime:** Python 3.9  
- **Role used:** `Lambda-S3-Processing-Role-Assignment3`  
- The code from `LambdaFunction.py` was uploaded to the Lambda editor.  
- The function filters out unnecessary rows and keeps only valid order data.

This automation removes the need for manual data cleaning and ensures consistent results every time a file is uploaded.

**Screenshot:**  
📸 *Figure 3.* Lambda Function Created  
![Screenshot3.png](./Screenshot/Screenshot3.png)


---

## 4. Configure the S3 Trigger ⚡

**Approach:**  
To make the data flow automatic, I set up an S3 trigger that runs the Lambda function whenever a new CSV file is uploaded to the `raw/` folder.

**Explanation:**  
- **Source:** S3  
- **Bucket:** `itcs6190.assignment3.801426261.kkim43`  
- **Event type:** All object create events  
- **Prefix:** `raw/`  
- **Suffix:** `.csv`  
- The trigger ensures the Lambda function only runs when new CSV files appear in the `raw/` folder.  

Once set up, uploading `Orders.csv` automatically starts the Lambda process, which saves the filtered result to `processed/`.

**Screenshot:**  
📸 *Figure 4.* Configured S3 Trigger  
![Screenshot4.png](./Screenshot/Screenshot4.png)


**Start Processing of Raw Data:**  
After setting up the trigger, I uploaded `Orders.csv` to the `raw/` folder.  
The Lambda function was automatically triggered, filtered the data, and saved the output file in the `processed/` folder.

**Screenshot:**  
📸 *Figure 5.* Processed CSV File in the `processed/` Folder on S3  
![Screenshot5.png](./Screenshot/Screenshot5.png)


--- 
**Start Processing of Raw Data**: Now upload the Orders.csv file into the `raw/` folder of the S3 Bucket. This will automatically trigger the Lambda function.
---

## 5. Create a Glue Crawler 🕸️

**Approach:**  
To make the processed data searchable in Athena, I used AWS Glue Crawler.  
The crawler scans the files in the `processed/` folder and automatically builds a data catalog table.

**Explanation:**  
- **Crawler name:** `orders_processed_crawler`  
- **Data source:** `s3://itcs6190.assignment3.801426261.kkim43/processed/`  
- **IAM Role:** `Glue-S3-Crawler-Role-Assignment3`  
- **Database:** `orders_db`  
- When the crawler runs, it scans the processed folder, detects the schema, and creates a table named `processed` under the `orders_db` database.

This allows Athena to recognize the dataset structure and run SQL queries on it.

**Screenshot:**  
📸 *Figure 6.* AWS Glue Crawler Configuration and Run Result  
![Screenshot6.png](./Screenshot/Screenshot6.png)



---

## 6. Query Data with Amazon Athena 🔍

**Approach:**  
After the Glue Crawler finished, the processed data became available in the Athena console.  
I used SQL queries in Athena to analyze the data directly from the `orders_db` database, which was automatically built from the processed CSV file.

**Explanation:**  
- **Data Source:** AwsDataCatalog  
- **Database:** `orders_db`  
- **Table:** `processed`  
- The SQL queries were used to calculate totals, trends, and insights from the filtered dataset.

**Queries executed:**
1. **Total Sales by Customer** — shows how much each customer spent in total.  
2. **Monthly Order Volume and Revenue** — counts monthly orders and revenue.  
3. **Order Status Dashboard** — groups orders by their status (`shipped` vs. `confirmed`).  
4. **Average Order Value (AOV) per Customer** — computes the average amount spent per order.  
5. **Top 10 Largest Orders in February 2025** — lists the highest-value orders from that month.

Each query successfully returned results using data from the `processed` table.

**Screenshot:**  
📸 *Figure 7.* Athena Query Execution Results and Output Files in the `enriched/` Folder  
![Screenshot7.png](./Screenshot/Screenshot7.png)


---
## 7. Launch the EC2 Web Server 🖥️

**Approach:**  
To show the Athena query results on a webpage, I launched an EC2 instance that runs a small Flask web app.  
This EC2 instance connects to Athena through the IAM role and displays query results in real time.

**Explanation:**  
1. Open the **EC2** console and click **Launch instance**.  
2. **Name:** `Athena-Dashboard-Server`  
3. **AMI:** `Amazon Linux 2023`  
4. **Instance Type:** `t3.micro` (Free tier eligible)  
5. **Key Pair:** Created a key pair named `assignment3-key.pem` and saved it securely.  
6. **Security Group:**  
   - **Rule 1:** SSH — Port 22 — Source: My IP  
   - **Rule 2:** Custom TCP — Port 5000 — Source: Anywhere (0.0.0.0/0)  
7. **IAM Role:** Selected `EC2-Athena-Dashboard-Role-Assignment3` for permissions.  
8. Launched the instance successfully and confirmed connection using SSH:
   ```bash
   ssh -i "assignment3-key.pem" ec2-user@<Your-Public-IP>
   ```
9. Installed Flask and Boto3, then ran the app to display Athena dashboard results.

---

## 8. Connect to Your EC2 Instance

**Approach:**  
After launching the EC2 instance, the next step is to connect to it using SSH.  
This allows you to install packages and set up the Flask app directly on the server.

**Explanation:**  
1. Go to the **EC2 dashboard** and select your running instance.  
2. Copy its **Public IPv4 address** from the instance details page.  
3. Open your terminal (or SSH client such as PuTTY) and connect using your key pair:
   ```bash
   ssh -i "assignment3-key.pem" ec2-user@18.188.227.169
   ```
4. Once connected, you should see a prompt like:
   ```
   [ec2-user@ip-18-188-227-169 ~]$
   ```
   This means your SSH connection was successful and you’re ready to configure the environment.

---

## 9. Set Up the Web Environment

**Approach:**  
After successfully connecting to the EC2 instance, the next step is to prepare the environment to run the Flask web app.  
This involves updating the system, installing Python, and adding the required libraries.

**Explanation:**  
Run the following commands one by one in your EC2 terminal:

1. **Update system packages**  
   Keep the OS and tools up to date before installing new software:
   ```bash
   sudo yum update -y
   ```

2. **Install Python and Pip**  
   These are needed to run the Flask application and manage dependencies:
   ```bash
   sudo yum install python3-pip -y
   ```

3. **Install Python libraries (Flask & Boto3)**  
   Flask will be used to host the web app, and Boto3 allows it to connect with AWS services:
   ```bash
   pip3 install Flask boto3
   ```

---

## 10. Create and Configure the Web Application

**Approach:**  
Now that the environment is ready, you’ll create a simple Flask web application that connects to Athena and displays query results dynamically.

**Explanation:**  

1. **Create the Flask app file**  
   Open a new Python file using the nano text editor:
   ```bash
   nano app.py
   ```

2. **Add your web application code**  
   Copy and paste the full content from your Python script (`EC2InstanceNANOapp.py`) into the editor.  
   This file includes Flask routes and Athena query logic that render your dashboard.

3. **Update the configuration values**  
   Before saving, make sure to edit the following variables at the top of your script:
   - `AWS_REGION`: your AWS region (e.g., `us-east-2`)  
   - `ATHENA_DATABASE`: your Glue database name (e.g., `orders_db`)  
   - `S3_OUTPUT_LOCATION`: your S3 path for Athena results (e.g., `s3://itcs6190.assignment3.801426261.kkim43/enriched/`)

4. **Save and exit nano**  
   Press `Ctrl + X`, then `Y`, and hit `Enter` to save your changes and return to the terminal.


---

## 11. Run the App and View Your Dashboard! 🚀

**Approach:**  
After setting up the Flask app, this step runs the web server so the Athena results can be viewed through a browser.

**Explanation:**  

1. **Start the Flask application**  
   Run the Python script to launch the web server:
   ```bash
   python3 app.py
   ```
   If everything is set up correctly, you’ll see an output similar to:
   ```
   * Running on http://0.0.0.0:5000/
   ```

2. **View the dashboard in your browser**  
   Open your web browser and enter the public IP address of your EC2 instance on port 5000:
   ```
   http://<YOUR_PUBLIC_IP_ADDRESS>:5000
   ```
   This will display the **Athena Orders Dashboard**, showing all query results in a clean, organized table layout.

**Screenshot:**  
📸 *Figure 8.* Final Webpage Result — Athena Dashboard on EC2  
![Screenshot8.png](./Screenshot/Screenshot8.png)

---

## Important Final Notes

* **Stopping the Server**: To stop the Flask application, return to your SSH terminal and press `Ctrl + C`.
* **Cost Management**: This setup uses free-tier services. To prevent unexpected charges, **stop or terminate your EC2 instance** from the AWS console when you are finished.






