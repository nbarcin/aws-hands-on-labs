# Introduction to Amazon Lambda — S3 Event-Driven Workflow

## 📌 Overview

This hands-on AWS lab demonstrates how to build a simple **event-driven serverless workflow** using **Amazon S3 and AWS Lambda**.

When an object is uploaded to a source S3 bucket, an S3 event automatically triggers a Lambda function. The Lambda function then copies the uploaded object to a destination S3 bucket.

### Architecture

```text
                    Upload Object
                         │
                         ▼
                ┌─────────────────┐
                │   S3 Source     │
                │     Bucket      │
                └────────┬────────┘
                         │
                    S3 Event
                         │
                         ▼
                ┌─────────────────┐
                │  AWS Lambda     │
                │ Python / Boto3  │
                └────────┬────────┘
                         │
                    Copy Object
                         │
                         ▼
                ┌─────────────────┐
                │ S3 Destination  │
                │     Bucket      │
                └─────────────────┘
                         
                         │
                         ▼
                ┌─────────────────┐
                │ Amazon CloudWatch│
                │      Logs       │
                └─────────────────┘
```

AWS S3 can send object-created events to Lambda, and the Lambda function can use its execution role to access other AWS resources.

---

## 🎯 Objectives

By completing this lab, I learned how to:

* Create Amazon S3 buckets
* Create an AWS Lambda function
* Configure a Lambda execution role
* Write Python code using Boto3
* Configure an S3 event trigger
* Connect Amazon S3 with AWS Lambda
* Automatically copy objects between S3 buckets
* Test an event-driven workflow
* Monitor Lambda execution using Amazon CloudWatch Logs
* Troubleshoot Lambda and S3 permission issues

---

## ☁️ AWS Services Used

| Service           | Purpose                                 |
| ----------------- | --------------------------------------- |
| Amazon S3         | Source and destination object storage   |
| AWS Lambda        | Serverless function that copies objects |
| AWS IAM           | Lambda execution permissions            |
| Amazon CloudWatch | Lambda logging and troubleshooting      |

---

## 🌎 Region

**AWS Region:** US East (N. Virginia) — `us-east-1`

The S3 buckets and Lambda function should be configured in the same AWS Region for this workflow.

---

# 🏗️ Lab Implementation

## 1. Create S3 Source Bucket

I created a source S3 bucket that receives uploaded objects.

Example:

```text
lambda-source-bucket-xxxx
```

> S3 bucket names must be globally unique.

---

## 2. Create S3 Destination Bucket

I created a second bucket to store the objects copied by Lambda.

Example:

```text
lambda-destination-bucket-xxxx
```

Using two separate buckets prevents the Lambda function from repeatedly triggering itself when it writes the copied object. AWS specifically recommends using two buckets for this type of workflow.

---

# 3. Create Lambda Function

### Function

```text
Function name: mylambdafunction
Runtime: Python 3.13
Architecture: x86_64
Region: us-east-1
```

Python 3.13 is currently a supported AWS Lambda runtime.

AWS Lambda includes Boto3 in the Python runtime, allowing the function to interact with AWS services.

---

# 4. Lambda Execution Role

The Lambda function uses an **IAM execution role**.

The role should provide:

* CloudWatch Logs permissions
* Permission to read objects from the source S3 bucket
* Permission to write objects to the destination S3 bucket

For the copy operation, the Lambda function needs at least:

```text
s3:GetObject
s3:PutObject
```

`CopyObject` requires read access to the source object and write access to the destination bucket.

AWS recommends following the principle of least privilege when configuring Lambda permissions.

---

# 5. Lambda Function Code

The Lambda function uses Boto3 to copy the uploaded object.

```python
import boto3

s3 = boto3.client("s3")

SOURCE_BUCKET = "YOUR-SOURCE-BUCKET"
DESTINATION_BUCKET = "YOUR-DESTINATION-BUCKET"


def lambda_handler(event, context):

    record = event["Records"][0]

    source_bucket = record["s3"]["bucket"]["name"]
    object_key = record["s3"]["object"]["key"]

    copy_source = {
        "Bucket": source_bucket,
        "Key": object_key
    }

    try:
        response = s3.copy_object(
            Bucket=DESTINATION_BUCKET,
            CopySource=copy_source,
            Key=object_key
        )

        print(f"Successfully copied: {object_key}")
        print(response)

        return {
            "statusCode": 200,
            "body": f"Successfully copied {object_key}"
        }

    except Exception as e:

        print(f"Error copying object: {str(e)}")

        raise
```

### Important

Replace:

```python
YOUR-SOURCE-BUCKET
```

and

```python
YOUR-DESTINATION-BUCKET
```

with your actual bucket names.

---

# 6. Configure S3 Trigger

The Lambda function was configured with an Amazon S3 trigger.

### Trigger configuration

```text
Source: S3
Bucket: Source S3 bucket
Event type: All object create events
```

When an object is created in the source bucket, S3 invokes the Lambda function.

When the trigger is configured through the Lambda console, AWS automatically adds the required resource-based permission allowing S3 to invoke the function.

---

# 7. Test the Event-Driven Workflow

I uploaded an image file to the source S3 bucket.

Example:

```text
image.jpeg
```

### Expected workflow

```text
image.jpeg
     │
     ▼
Source S3 Bucket
     │
     │ S3 ObjectCreated Event
     ▼
AWS Lambda
     │
     │ boto3.copy_object()
     ▼
Destination S3 Bucket
     │
     ▼
image.jpeg
```

The uploaded image should appear automatically in the destination bucket.

---

# 8. CloudWatch Logs

Amazon CloudWatch Logs can be used to verify Lambda execution and troubleshoot errors.

Example log messages:

```text
Successfully copied: image.jpeg
```

If the copy fails, CloudWatch Logs can help identify problems such as:

* Incorrect bucket names
* Missing IAM permissions
* Incorrect object keys
* S3 trigger configuration issues
* Lambda execution errors

Lambda functions can write execution logs to CloudWatch when the execution role has the required logging permissions.

---

# ✅ Validation

The lab was successfully completed when:

* [x] Source S3 bucket was created
* [x] Destination S3 bucket was created
* [x] Lambda function was created
* [x] Lambda execution role was configured
* [x] Lambda Python code was deployed
* [x] S3 trigger was configured
* [x] Test image was uploaded
* [x] Lambda function was triggered
* [x] Image was copied to destination bucket
* [x] CloudWatch Logs were verified

---

# 🧠 Key Concepts Learned

### Serverless Computing

AWS Lambda runs code without requiring the user to provision or manage servers.

### Event-Driven Architecture

An event in one AWS service can automatically trigger an action in another service.

```text
S3 Event → Lambda → S3
```

### IAM Execution Role

The Lambda execution role determines which AWS resources and APIs the function can access.

### Boto3

Boto3 is the AWS SDK for Python and allows Python applications and Lambda functions to interact with AWS services.

### CloudWatch Logs

CloudWatch Logs provides visibility into Lambda executions and is useful for troubleshooting.

### Least Privilege

IAM permissions should be limited to the actions and resources required by the application.

---

# 🔐 Security Considerations

For a production implementation, permissions should be restricted to the specific buckets and actions required.

Instead of broad permissions such as:

```text
s3:*
```

use narrowly scoped permissions such as:

```text
s3:GetObject
s3:PutObject
```

and restrict them to the appropriate S3 resources.

AWS recommends least-privilege permissions for Lambda execution roles.

---

# 📸 Screenshots

The following screenshots document the implementation:

1. S3 source and destination buckets
2. Lambda function configuration
3. Lambda execution role
4. Lambda Python code
5. S3 trigger configuration
6. Source bucket with uploaded object
7. Destination bucket containing copied object
8. CloudWatch Lambda logs

---

# 🧹 Cleanup

To avoid unnecessary AWS charges, delete the resources after completing the lab:

* Delete the test object
* Empty and delete the source S3 bucket
* Empty and delete the destination S3 bucket
* Delete the Lambda function
* Delete the IAM role if it was created only for this lab

---

# 📚 AWS Documentation

* AWS Lambda with Amazon S3
* AWS Lambda Python runtimes
* Lambda execution roles and permissions
* Amazon S3 event notifications

Official AWS documentation was used to validate the current Lambda/S3 workflow and permission requirements.
