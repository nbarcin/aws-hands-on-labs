# AWS Fundamentals Lab: EC2, S3, IAM & Billing

## Overview

This hands-on AWS lab demonstrates the fundamentals of **cloud computing, identity and access management, object storage, compute services, and cost management** using Amazon Web Services (AWS).

The lab focuses on creating and configuring AWS resources, connecting an EC2 instance to S3 using an IAM role, and exploring AWS Billing and Cost Management tools.

## Objectives

By completing this lab, I practiced how to:

* Create and apply an **IAM role** to an AWS service
* Create an **Amazon S3 bucket**
* Upload objects to an S3 bucket
* Launch and manage an **Amazon EC2 instance**
* Connect to an EC2 instance
* Use an IAM role to access S3 from EC2
* Verify AWS permissions using the AWS CLI
* Explore **AWS Billing and Cost Management**
* Review AWS costs using **Cost Explorer**
* Terminate AWS resources to avoid unnecessary charges

## AWS Services Used

| Service                           | Purpose                                    |
| --------------------------------- | ------------------------------------------ |
| **Amazon EC2**                    | Virtual server / compute resource          |
| **Amazon S3**                     | Object storage                             |
| **AWS IAM**                       | Identity and access management             |
| **AWS Billing & Cost Management** | Cost monitoring and analysis               |
| **AWS CLI**                       | Command-line interaction with AWS services |

## Architecture

```text
                   AWS Account
                       |
          +------------+-------------+
          |                          |
        IAM                         S3
          |                    +-------------+
          |                    | S3 Bucket   |
          |                    |             |
          |                    | test.txt    |
          |                    +-------------+
          |
    IAM Role
          |
          | S3 ReadOnlyAccess
          |
        EC2
   +-------------+
   | Amazon Linux|
   | 2023        |
   |             |
   | AWS CLI     |
   +-------------+
          |
          | aws s3 ls
          |
          v
       Amazon S3
```

## Lab Environment

* **Cloud Provider:** AWS
* **Operating System:** Amazon Linux 2023
* **Region:** `us-east-1` (N. Virginia) or another selected AWS Region
* **EC2 Instance:** Micro instance type appropriate for the account's current free-tier/credit eligibility
* **Access:** AWS Management Console + EC2 Instance Connect / SSH
* **CLI:** AWS CLI

> **Note:** AWS pricing and Free Tier eligibility can change. Always verify the current pricing and Free Tier terms before launching resources.

---

# Lab Steps

## 1. Create an IAM Role

I created an IAM role that allows an EC2 instance to access Amazon S3.

### Configuration

**Trusted entity:**

* AWS Service

**Use case:**

* EC2

**Permission policy:**

* `AmazonS3ReadOnlyAccess`

**Example role name:**

```text
EC2-S3-ReadOnly-Role
```

### Purpose

The IAM role allows the EC2 instance to access S3 without storing AWS access keys directly on the server.

This demonstrates the AWS security best practice of using **IAM roles and temporary credentials** for AWS services.

---

## 2. Create an Amazon S3 Bucket

I created an S3 bucket and uploaded a test file.

### Example

```text
Bucket name:
lab-bucket-unique-name
```

### Actions

1. Open Amazon S3
2. Create a bucket
3. Select the desired AWS Region
4. Keep the required security settings enabled
5. Create the bucket
6. Open the bucket
7. Upload a test file

Example:

```text
test.txt
```

---

## 3. Launch an Amazon EC2 Instance

I launched an EC2 instance using Amazon Linux 2023.

### Configuration

* **AMI:** Amazon Linux 2023
* **Instance Type:** Micro-sized instance appropriate for the account
* **Key Pair:** RSA `.pem`
* **SSH:** My IP
* **HTTP:** Enabled for the lab
* **IAM Instance Profile:** `EC2-S3-ReadOnly-Role`

### Security Considerations

SSH access was restricted to my current IP address rather than allowing SSH from the entire internet.

For production environments, security groups should follow the principle of least privilege and expose only the required ports.

---

## 4. Connect to the EC2 Instance

After the instance reached the **Running** state, I connected to it using EC2 Instance Connect.

I verified the operating system with:

```bash
cat /etc/os-release
```

I also verified the AWS CLI:

```bash
aws --version
```

---

## 5. Verify IAM Role and S3 Access

From the EC2 instance, I used the AWS CLI to verify access to Amazon S3.

### List S3 buckets

```bash
aws s3 ls
```

The command successfully returned the accessible S3 resources.

### List objects in the bucket

```bash
aws s3 ls s3://YOUR-BUCKET-NAME
```

This demonstrated that the EC2 instance could access S3 using the attached IAM role.

### Important Security Concept

No AWS access key or secret access key was manually configured on the EC2 instance.

The EC2 instance received permissions through its **IAM instance role**.

---

# 6. Explore AWS Billing and Cost Management

I explored the AWS Billing and Cost Management dashboard to understand how AWS resources and services contribute to cloud costs.

### Areas Reviewed

* Billing dashboard
* Bills
* Cost Explorer
* Service-level cost information
* Current and estimated costs

### Cost Explorer

I reviewed service-level information to understand how AWS usage can be monitored and analyzed.

Example services:

```text
Amazon EC2
Amazon S3
```

Cost monitoring is an important part of cloud administration because improperly configured or unused resources can generate unexpected charges.

---

# Skills Demonstrated

This lab demonstrates practical experience with:

* AWS Management Console
* Amazon EC2
* Amazon S3
* AWS IAM
* IAM Roles
* IAM Policies
* AWS CLI
* Linux command line
* EC2 Instance Connect
* Security Groups
* SSH security
* Cloud security fundamentals
* Least privilege
* AWS Billing
* Cost Explorer
* Cloud resource cleanup

---

# Key Takeaways

### IAM Roles

IAM roles provide AWS services with temporary permissions without requiring long-term credentials to be stored on the instance.

### Amazon S3

S3 provides scalable object storage and can be accessed through the AWS Console and AWS CLI.

### Amazon EC2

EC2 provides virtual compute capacity that can be configured with different operating systems, instance types, networking, storage, and security settings.

### AWS CLI

The AWS CLI provides a practical way to interact with AWS services from the command line.

### Cloud Cost Management

Monitoring AWS costs is an essential cloud administration skill. Resources should be monitored and removed when they are no longer required.



# Evidence / Screenshots

The repository includes screenshots documenting the major stages of the lab.

Recommended evidence:

1. IAM role configuration
2. IAM role permissions
3. S3 bucket
4. Uploaded S3 object
5. EC2 instance running
6. EC2 security group configuration
7. IAM instance profile attached to EC2
8. EC2 terminal connection
9. AWS CLI S3 verification
10. Billing / Cost Explorer

> Sensitive information such as account IDs, public IP addresses, private keys, email addresses, and other credentials should be removed or blurred before publishing screenshots.

---

# Conclusion

This lab provided hands-on experience with several foundational AWS services and demonstrated how **compute, storage, identity, security, and cost management** work together in a cloud environment.

The most important concept demonstrated in this lab was using an **IAM role to provide an EC2 instance with controlled access to Amazon S3 without storing long-term AWS credentials on the server**.

---

## Author

**Nihal Barcin**

M.S. Computer Science | AI & Cloud Computing

Areas of Interest:

* Cloud Computing
* Artificial Intelligence
* Machine Learning
* Cloud Security
* Data Analysis
* AWS
* LLMs and AI Agents
