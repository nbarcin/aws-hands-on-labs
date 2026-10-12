# AWS S3 Static Website Hosting | AWS CLI, IAM & Bash Automation

## Project Overview

This project demonstrates how to deploy and update a static website on Amazon S3 using the AWS Command Line Interface (AWS CLI). It also covers IAM user and policy management, AWS Systems Manager Session Manager, and Bash scripting to automate website deployments.

The lab uses a Café and Bakery website as an example and demonstrates how to upload website files, enable static website hosting, and publish updates through a reusable deployment script.

## Architecture

```text
AWS Systems Manager
        |
        v
Amazon Linux EC2 Instance
        |
        | AWS CLI
        v
Amazon S3 Bucket
        |
        +-- index.html
        +-- css/
        +-- images/
        |
        v
Static Website Endpoint
        |
        v
Website Visitors
```

## AWS Services and Technologies

* **Amazon EC2:** Provides the Linux environment for executing AWS CLI commands.
* **AWS Systems Manager Session Manager:** Enables browser-based access to the EC2 instance.
* **AWS Identity and Access Management (IAM):** Manages users, roles, and permissions.
* **Amazon S3:** Stores website files and hosts the static website.
* **AWS CLI:** Automates AWS resource management and file uploads.
* **Bash:** Creates a repeatable website deployment script.
* **Linux:** Provides the command-line environment for administration and automation.
* **HTML, CSS, and images:** Website content and assets.

## Lab Objectives

* Connect to an Amazon Linux EC2 instance using Session Manager.
* Configure the AWS CLI for the lab environment.
* Create an S3 bucket in the `us-west-2` Region.
* Create an IAM user and attach an AWS managed policy.
* Configure S3 bucket permissions and object ownership.
* Enable static website hosting.
* Upload website files using the AWS CLI.
* Automate website updates with a Bash script.
* Improve deployment efficiency using `aws s3 sync`.

## Implementation Steps

### 1. Connect to EC2 Using Session Manager

Connected to the Amazon Linux EC2 instance through AWS Systems Manager Session Manager.

Switched to the `ec2-user` account and verified the working directory.

```bash
sudo su -l ec2-user
pwd
```

### 2. Configure the AWS CLI

Configured the AWS CLI with the credentials provided by the training environment.

```bash
aws configure
```

Configuration used:

* Default Region: `us-west-2`
* Default output format: `json`

**Security note:** Credentials should be handled securely and never committed to GitHub.

### 3. Create an S3 Bucket

Created an S3 bucket using the AWS CLI.

```bash
aws s3api create-bucket \
  --bucket YOUR-UNIQUE-BUCKET-NAME \
  --region us-west-2 \
  --create-bucket-configuration LocationConstraint=us-west-2
```

S3 bucket names must be globally unique.

### 4. Create an IAM User and Attach a Policy

Created an IAM user for the lab:

```bash
aws iam create-user --user-name awsS3user
```

Identified the AWS managed policy that provides full S3 access:

```bash
aws iam list-policies \
  --query "Policies[?contains(PolicyName,'S3')]"
```

Attached the policy:

```bash
aws iam attach-user-policy \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess \
  --user-name awsS3user
```

Verified the attached policies:

```bash
aws iam list-attached-user-policies \
  --user-name awsS3user
```

**Important:** Attaching a policy requires the calling identity to have the appropriate IAM permissions. An EC2 instance role may not be authorized to perform `iam:AttachUserPolicy`.

For production environments, use least-privilege policies rather than granting full S3 access.

### 5. Extract the Website Files

Extracted the supplied static website files.

```bash
cd ~/sysops-activity-files
tar xvzf static-website-v2.tar.gz
cd static-website
ls
```

The directory contains the website's `index.html` file and supporting `css` and `images` directories.

### 6. Enable Static Website Hosting

Configured the S3 bucket to use `index.html` as its index document.

```bash
aws s3 website s3://YOUR-UNIQUE-BUCKET-NAME/ \
  --index-document index.html
```

### 7. Upload Website Files to S3

Uploaded the website files recursively:

```bash
aws s3 cp \
  /home/ec2-user/sysops-activity-files/static-website/ \
  s3://YOUR-UNIQUE-BUCKET-NAME/ \
  --recursive
```

Verified the uploaded objects:

```bash
aws s3 ls s3://YOUR-UNIQUE-BUCKET-NAME/
```

The lab also demonstrates using `--acl public-read` for public website access. This requires compatible bucket Object Ownership settings and appropriate permissions. Public access can also be blocked by S3 Block Public Access settings or organization policies.

### 8. Automate Website Updates with Bash

Created a reusable deployment script:

```bash
cd ~
touch update-website.sh
vi update-website.sh
```

Added the following content, replacing the bucket name with the actual bucket name:

```bash
#!/bin/bash

aws s3 cp \
  /home/ec2-user/sysops-activity-files/static-website/ \
  s3://YOUR-UNIQUE-BUCKET-NAME/ \
  --recursive
```

Made the script executable:

```bash
chmod +x update-website.sh
```

Executed the deployment script:

```bash
./update-website.sh
```

This makes it easier to publish website changes without repeatedly entering the full AWS CLI command.

### 9. Improve Deployment Efficiency with S3 Sync

The lab introduces `aws s3 sync` as an alternative to repeatedly copying all files.

```bash
aws s3 sync \
  /home/ec2-user/sysops-activity-files/static-website/ \
  s3://YOUR-UNIQUE-BUCKET-NAME/
```

Unlike a recursive copy, `sync` compares source and destination files and normally transfers only new or changed files.

The command can reduce unnecessary uploads when most website files remain unchanged.

## Security Considerations

This lab demonstrates IAM permissions and public S3 website hosting in a controlled training environment.

For real-world deployments:

* Follow the principle of least privilege.
* Avoid hardcoding AWS access keys in scripts or source code.
* Prefer IAM roles and temporary credentials when possible.
* Keep S3 Block Public Access enabled unless public access is explicitly required.
* Consider Amazon CloudFront with a private S3 origin for public website delivery.
* Avoid granting `AmazonS3FullAccess` when narrower bucket-level permissions are sufficient.
* Remove lab resources and credentials when they are no longer needed.

## Key Learnings

* Managed an EC2 Linux environment through Systems Manager Session Manager.
* Used AWS CLI commands to create and inspect AWS resources.
* Practiced IAM users, managed policies, and permission troubleshooting.
* Deployed static website content to Amazon S3.
* Worked with S3 static website hosting and object permissions.
* Automated repetitive deployment tasks using Bash.
* Compared `aws s3 cp` and `aws s3 sync`.
* Applied security and least-privilege concepts to cloud administration.

## Conclusion

This project demonstrates a practical AWS workflow for deploying and updating a static website using Amazon S3, AWS CLI, IAM, and Bash scripting.

It provides hands-on experience with cloud resource management, access control, Linux administration, and deployment automation—skills relevant to cloud engineering, cloud operations, and DevOps workflows.
