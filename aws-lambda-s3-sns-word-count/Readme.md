# AWS Lambda + S3 + SNS Word Count

## Overview

This project demonstrates an event-driven, serverless AWS workflow using **Amazon S3, AWS Lambda, Amazon SNS, IAM, and Amazon CloudWatch**.

When a text file is uploaded to an Amazon S3 bucket, an S3 event automatically invokes an AWS Lambda function. The Lambda function reads the text file, counts the number of words, and publishes the result to an Amazon SNS topic. SNS then sends the result to a subscribed email address.

### Architecture

```text
                    Upload .txt File
                          │
                          ▼
                  ┌───────────────┐
                  │   Amazon S3   │
                  │     Bucket    │
                  └───────┬───────┘
                          │
                    S3 Event Trigger
                          │
                          ▼
                  ┌───────────────┐
                  │ AWS Lambda    │
                  │ WordCount     │
                  │ Function      │
                  └───────┬───────┘
                          │
                    Count Words
                          │
                          ▼
                  ┌───────────────┐
                  │ Amazon SNS    │
                  │    Topic      │
                  └───────┬───────┘
                          │
                          ▼
                       Email
```

---

## AWS Services Used

* **Amazon S3** – Stores the text files and generates an event when a file is uploaded.
* **AWS Lambda** – Reads the uploaded file and counts the words.
* **Amazon SNS** – Sends the word-count result by email.
* **AWS IAM** – Provides permissions for Lambda to access AWS services.
* **Amazon CloudWatch** – Stores Lambda execution logs and helps monitor the function.

---

## IAM Role

The Lambda function uses the existing lab IAM role:

```text
LambdaAccessRole
```

The role provides the permissions required for the lab, including access to:

* Amazon S3
* Amazon SNS
* Amazon CloudWatch Logs
* Amazon CloudWatch

---

## Lambda Function

### Function Name

```text
WordCountFunction
```

### Runtime

```text
Python
```

### Functionality

The Lambda function:

1. Receives an event from Amazon S3.
2. Identifies the bucket and uploaded file.
3. Reads the text file from S3.
4. Counts the words using Python.
5. Creates the notification message.
6. Publishes the result to Amazon SNS.
7. Returns the result.

### Word Counting Logic

The word count is calculated using:

```python
word_count = len(text.split())
```

The notification message is formatted as:

```text
The word count in the <textFileName> file is nnn.
```

The SNS email subject is:

```text
Word Count Result
```

---

## Lambda Code

```python
import boto3

s3 = boto3.client('s3')
sns = boto3.client('sns')

SNS_TOPIC_ARN = "YOUR_SNS_TOPIC_ARN"


def lambda_handler(event, context):

    # Get bucket name and file name from the S3 event
    bucket_name = event['Records'][0]['s3']['bucket']['name']
    file_key = event['Records'][0]['s3']['object']['key']

    # Read the file from S3
    response = s3.get_object(
        Bucket=bucket_name,
        Key=file_key
    )

    # Read the file contents
    text = response['Body'].read().decode('utf-8')

    # Count the words
    word_count = len(text.split())

    # Create the message
    message = f"The word count in the {file_key} file is {word_count}."

    # Send the result through SNS
    sns.publish(
        TopicArn=SNS_TOPIC_ARN,
        Subject="Word Count Result",
        Message=message
    )

    return {
        'statusCode': 200,
        'body': message
    }
```

> **Note:** The actual SNS Topic ARN is not included in this README to avoid exposing account-specific information.

---

## S3 Event Trigger

The Lambda function is automatically invoked when an object is created in the S3 bucket.

The workflow is:

```text
File Upload
     ↓
S3 Object Created Event
     ↓
AWS Lambda
```

For testing, the following text files were uploaded:

```text
sample1.txt
sample2.txt
sample3.txt
```

Each file contained a different number of words.

---

## SNS Notification

The Lambda function publishes the result to the SNS topic:

```text
WordCountResults
```

The email subject is:

```text
Word Count Result
```

Example email:

```text
The word count in the sample1.txt file is 3.
```

---

## Testing

The solution was tested by uploading multiple text files with different word counts.

### Test Case 1

File:

```text
sample1.txt
```

Content:

```text
Hello AWS Lambda
```

Expected result:

```text
The word count in the sample1.txt file is 3.
```

### Test Case 2

File:

```text
sample2.txt
```

Content:

```text
AWS Lambda can process files automatically.
```

Expected result:

```text
The word count in the sample2.txt file is 6.
```


---

## Monitoring

AWS CloudWatch Logs were used to verify Lambda executions and troubleshoot potential errors.

Successful execution can be verified through:

```text
AWS Lambda
    ↓
Monitor
    ↓
CloudWatch Logs
```

---

## Key AWS Concepts Demonstrated

This lab demonstrates several important AWS concepts:

* Serverless computing
* Event-driven architecture
* AWS Lambda
* Amazon S3 event notifications
* Amazon SNS notifications
* IAM roles and permissions
* CloudWatch monitoring
* Python with AWS SDK (`boto3`)
* S3 object retrieval
* Automated cloud workflows

---

## Project Architecture

```text
                    ┌─────────────────┐
                    │   Local Machine │
                    │   sample1.txt   │
                    │   sample2.txt   │
                    │   sample3.txt   │
                    └────────┬────────┘
                             │
                          Upload
                             │
                             ▼
                    ┌─────────────────┐
                    │   Amazon S3     │
                    │     Bucket      │
                    └────────┬────────┘
                             │
                       S3 Event
                             │
                             ▼
                    ┌─────────────────┐
                    │   AWS Lambda    │
                    │ WordCountFunction│
                    └────────┬────────┘
                             │
                       Count Words
                             │
                             ▼
                    ┌─────────────────┐
                    │   Amazon SNS    │
                    │ WordCountResults│
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │      Email      │
                    │ Word Count Result│
                    └─────────────────┘
```


## Result

The completed solution successfully demonstrates an automated serverless workflow:

```text
Upload File
     ↓
Amazon S3
     ↓
S3 Event
     ↓
AWS Lambda
     ↓
Word Count
     ↓
Amazon SNS
     ↓
Email Notification
```

This project demonstrates how AWS managed services can be combined to create an automated, event-driven application without managing servers.
