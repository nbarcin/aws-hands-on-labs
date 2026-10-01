# AWS Lambda – Sales Analysis Report

## Overview

This lab demonstrates how to build a serverless sales analysis workflow using **AWS Lambda** and several AWS services.

The solution consists of two Lambda functions:

* `salesAnalysisReportDataExtractor`
* `salesAnalysisReport`

The first Lambda function connects to a MySQL database and extracts sales data. The second Lambda function retrieves the database information, invokes the data extractor, formats the report, and publishes the result through Amazon SNS.

The final solution uses a scheduled EventBridge rule to automatically generate the sales report.

---

## AWS Services Used

* **AWS Lambda** – Serverless compute
* **AWS Lambda Layers** – Reusable Python dependency
* **AWS IAM** – Roles and permissions
* **Amazon VPC** – Network connectivity
* **Amazon EC2** – MySQL database / application environment
* **AWS Systems Manager Parameter Store** – Database configuration
* **Amazon SNS** – Email notifications
* **Amazon EventBridge** – Scheduled Lambda execution
* **Amazon CloudWatch Logs** – Monitoring and troubleshooting
* **AWS CLI** – Lambda deployment

---

## Architecture

```text
                    EventBridge
                        |
                        | Scheduled Trigger
                        v
              +-----------------------+
              |   salesAnalysisReport |
              |       Lambda          |
              +-----------+-----------+
                          |
             Invokes data extractor
                          |
                          v
        +----------------------------------+
        | salesAnalysisReportDataExtractor |
        |             Lambda               |
        +----------------+-----------------+
                         |
                         | PyMySQL
                         |
                         v
                 +---------------+
                 | MySQL / EC2   |
                 | Cafe Database |
                 +---------------+

salesAnalysisReport
        |
        | Retrieves DB parameters
        v
Parameter Store

salesAnalysisReport
        |
        | Publish report
        v
       SNS
        |
        v
      Email
```

---

# 1. IAM Roles and Permissions

I reviewed the IAM roles used by the Lambda functions.

### salesAnalysisReportRole

This role allows the main Lambda function to:

* Publish messages to Amazon SNS
* Read parameters from Systems Manager Parameter Store
* Write logs to CloudWatch
* Invoke another Lambda function

Policies observed in the lab:

* `AmazonSNSFullAccess`
* `AmazonSSMReadOnlyAccess`
* `AWSLambdaBasicRunRole`
* `AWSLambdaRole`

The role trusts `lambda.amazonaws.com`.

### salesAnalysisReportDERole

This role is used by the data extractor Lambda function.

Policies include:

* `AWSLambdaBasicRunRole`
* `AWSLambdaVPCAccessRunRole`

The VPC access policy allows the Lambda function to manage the network interfaces required for VPC connectivity.

---

# 2. Creating a Lambda Layer

I created a Lambda Layer named:

```text
pymysqlLibrary
```

The layer contains the **PyMySQL** library required by the data extractor Lambda function.

### Why use a Lambda Layer?

Instead of packaging the PyMySQL dependency directly with every Lambda function, the dependency can be maintained separately and reused.

Configuration:

```text
Layer name: pymysqlLibrary
Runtime: Python 3.9
Package: pymysql-v3.zip
```

---

# 3. Creating the Data Extractor Lambda

Lambda function:

```text
salesAnalysisReportDataExtractor
```

Runtime:

```text
Python 3.9
```

IAM Role:

```text
salesAnalysisReportDERole
```

Handler:

```text
salesAnalysisReportDataExtractor.lambda_handler
```

The `pymysqlLibrary` layer was attached to the function.

The function receives the following database parameters through the Lambda event:

```json
{
  "dbUrl": "...",
  "dbName": "...",
  "dbUser": "...",
  "dbPassword": "..."
}
```

The function then connects to the MySQL database and retrieves sales information.

---

# 4. VPC Configuration

The data extractor Lambda requires network access to the MySQL database running on an EC2 instance.

The Lambda function was configured with:

```text
VPC: Cafe VPC
Subnet: Cafe Public Subnet 1
Security Group: CafeSecurityGroup
```

This configuration allows the Lambda function to communicate with the database inside the VPC.

---

# 5. Troubleshooting Lambda Timeout

The first test of the data extractor Lambda failed with a timeout:

```text
Task timed out after 3.00 seconds
```

The function was attempting to connect to the MySQL database but could not establish the connection successfully.

### Root Cause

The MySQL database uses port:

```text
3306
```

The EC2 security group's inbound rules needed to allow the Lambda function's traffic to the MySQL database.

After correcting the security group configuration, the Lambda test succeeded.

Successful response:

```json
{
  "statusCode": 200,
  "body": []
}
```

The empty body was expected because the database did not yet contain order data.

---

# 6. Testing with Database Data

Orders were placed through the café application to populate the database.

The Lambda function was then tested again.

Example response:

```json
{
  "statusCode": 200,
  "body": [
    {
      "product_group_number": 1,
      "product_group_name": "Pastries",
      "product_id": 1,
      "product_name": "Croissant",
      "quantity": 1
    },
    {
      "product_group_number": 2,
      "product_group_name": "Drinks",
      "product_id": 8,
      "product_name": "Hot Chocolate",
      "quantity": 2
    }
  ]
}
```

This confirmed that the Lambda function could successfully connect to the database and retrieve sales data.

---

# 7. Amazon SNS Configuration

I created an SNS topic:

```text
salesAnalysisReportTopic
```

Display name:

```text
SARTopic
```

An email subscription was then added to the SNS topic.

The subscription was confirmed through the AWS confirmation email.

The SNS topic is responsible for delivering the generated sales report to the subscribed email address.

---

# 8. Creating the Main Lambda Function

Lambda function:

```text
salesAnalysisReport
```

Runtime:

```text
Python 3.9
```

The function was deployed using the AWS CLI.

Example deployment command:

```bash
aws lambda create-function \
  --function-name salesAnalysisReport \
  --runtime python3.9 \
  --zip-file fileb://salesAnalysisReport-v2.zip \
  --handler salesAnalysisReport.lambda_handler \
  --region <region> \
  --role <salesAnalysisReportRoleARN>
```

---

# 9. Environment Variable

The main Lambda function uses an environment variable to identify the SNS topic.

```text
Key: topicARN
Value: <SNS Topic ARN>
```

This allows the Lambda function to publish the generated report to the correct SNS topic.

---

# 10. Testing the Main Lambda

The Lambda function was tested using a test event named:

```text
SARTestEvent
```

The execution completed successfully.

Expected response:

```json
{
  "statusCode": 200,
  "body": "\"Sale Analysis Report sent.\""
}
```

The generated sales report was delivered through Amazon SNS to the subscribed email address.

---

# 11. Scheduled Execution with EventBridge

The final step was configuring an EventBridge schedule to automatically invoke the Lambda function.

Rule:

```text
salesAnalysisReportDailyTrigger
```

Description:

```text
Initiates report generation on a daily basis
```

Schedule type:

```text
Schedule expression
```

The schedule uses an AWS cron expression.

Important:

AWS EventBridge cron expressions use **UTC**.

Example format:

```text
cron(Minutes Hours Day-of-month Month Day-of-week Year)
```

The production schedule should be configured for Monday through Saturday at 8 PM according to the required time zone.

---

# 12. CloudWatch Logs

CloudWatch Logs were used to troubleshoot the Lambda timeout.

Important Lambda log entries include:

```text
START
END
REPORT
```

The logs provide information about:

* Lambda execution
* Execution duration
* Memory usage
* Errors
* Timeout conditions

This was useful for identifying the initial database connectivity problem.

---

# 13. Key Learning Outcomes

Through this lab, I practiced:

* Creating and configuring AWS Lambda functions
* Creating and attaching Lambda Layers
* Managing IAM roles and permissions
* Connecting Lambda to a VPC
* Connecting Lambda to a MySQL database
* Using Systems Manager Parameter Store
* Configuring Amazon SNS email notifications
* Deploying Lambda functions using AWS CLI
* Configuring Lambda environment variables
* Using EventBridge scheduled triggers
* Troubleshooting Lambda using CloudWatch Logs
* Understanding Lambda-to-Lambda invocation

---

## Troubleshooting Summary

| Problem                 | Cause                                        | Solution                                           |
| ----------------------- | -------------------------------------------- | -------------------------------------------------- |
| Lambda timeout          | Database connection could not be established | Updated security group rules for MySQL port 3306   |
| Empty report data       | Database had no orders                       | Placed orders through the café application         |
| Lambda dependency issue | PyMySQL was not included in function package | Created and attached `pymysqlLibrary` Lambda Layer |
| Scheduled execution     | Lambda needed automatic invocation           | Configured EventBridge schedule                    |

---

## AWS Skills Demonstrated

```text
AWS Lambda
IAM
Lambda Layers
VPC
EC2
Security Groups
Systems Manager Parameter Store
Amazon SNS
EventBridge
CloudWatch
AWS CLI
Python
MySQL
Serverless Architecture
Troubleshooting
```

---

## Result

The completed solution demonstrates a serverless sales reporting workflow where:

```text
EventBridge
     ↓
salesAnalysisReport
     ↓
salesAnalysisReportDataExtractor
     ↓
MySQL Database
     ↓
Sales Data
     ↓
Amazon SNS
     ↓
Email Report
```

The lab demonstrates practical experience with **AWS serverless architecture, IAM, networking, monitoring, automation, and troubleshooting**.
