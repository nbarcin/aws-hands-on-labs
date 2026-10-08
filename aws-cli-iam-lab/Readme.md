# AWS CLI and IAM Configuration Lab

## Overview

This lab demonstrates how to install and configure the **AWS Command Line Interface (AWS CLI)** on a Red Hat Linux instance and use the CLI to interact with **AWS Identity and Access Management (IAM)**.

The lab focuses on:

* Installing AWS CLI on a Linux EC2 instance
* Configuring AWS CLI credentials and default region
* Testing AWS account connectivity
* Querying IAM users using AWS CLI
* Listing customer-managed IAM policies
* Retrieving an IAM policy version in JSON format
* Saving IAM policy information to a local JSON file
* Working with AWS services through the command line instead of relying on the AWS Management Console

---

## Technologies and Services

* **Amazon EC2**
* **Red Hat Linux**
* **AWS CLI v2**
* **AWS IAM**
* **Linux Command Line**
* **SSH**
* **JSON**

---

## Lab Architecture

```text
┌─────────────────────────────┐
│       AWS Account           │
│                             │
│  ┌───────────────────────┐  │
│  │     IAM               │  │
│  │                       │  │
│  │  awsstudent           │  │
│  │  lab_policy            │  │
│  └───────────┬───────────┘  │
│              │              │
│              │ AWS API       │
│              ▼              │
│  ┌───────────────────────┐  │
│  │ EC2 Instance          │  │
│  │ Red Hat Linux         │  │
│  │                       │  │
│  │ AWS CLI v2             │  │
│  └───────────────────────┘  │
└─────────────────────────────┘
```

---

# Task 1 — Connect to the Linux Instance

An SSH connection was established to the Red Hat Linux EC2 instance.

Example:

```bash
ssh -i <key-file.pem> <user>@<public-ip>
```

> **Security:** Private SSH keys and public IP addresses used for temporary labs should not be committed to GitHub.

---

# Task 2 — Install AWS CLI on Red Hat Linux

The AWS CLI v2 installer was downloaded using `curl`.

### Download AWS CLI

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```

### Extract the installer

```bash
unzip -u awscliv2.zip
```

### Install AWS CLI

```bash
sudo ./aws/install
```

### Verify the installation

```bash
aws --version
```

Example:

```text
aws-cli/2.x.x Python/3.x.x Linux/...
```

The exact version may vary because AWS CLI versions are updated over time.

### Test AWS CLI

```bash
aws help
```

Press `q` to exit the help page.

---

# Task 3 — Review IAM Configuration

The lab initially used the AWS Management Console to observe the IAM configuration.

The following IAM information was reviewed:

* IAM user
* IAM permissions
* `lab_policy`
* Policy JSON document
* Access key configuration

The `lab_policy` customer-managed policy controls which AWS services and actions the lab user can access.

> **Security Note:** Access keys and secret access keys are sensitive credentials. They must never be uploaded to GitHub.

---

# Task 4 — Configure AWS CLI

The AWS CLI was configured using:

```bash
aws configure
```

The following configuration values were entered:

```text
AWS Access Key ID: <ACCESS_KEY_ID>
AWS Secret Access Key: <SECRET_ACCESS_KEY>
Default region name: us-west-2
Default output format: json
```

### Verify AWS CLI configuration

The configuration can be checked with:

```bash
aws configure list
```

> Never include actual credentials in this README or commit AWS credential files to GitHub.

---

# Task 5 — Test IAM Access Using AWS CLI

The IAM configuration was tested by listing IAM users:

```bash
aws iam list-users
```

A successful command returns JSON containing the IAM users available in the account.

Example structure:

```json
{
    "Users": [
        {
            "Path": "/",
            "UserName": "example-user",
            "Arn": "arn:aws:iam::ACCOUNT-ID:user/example-user"
        }
    ]
}
```

---

# Activity 1 — Retrieve IAM Policy Using AWS CLI

The challenge was to retrieve the `lab_policy` IAM policy using the AWS CLI without relying on the AWS Management Console.

## Step 1 — List Customer-Managed Policies

Because `lab_policy` is a customer-managed policy, the policy scope was set to `Local`:

```bash
aws iam list-policies --scope Local
```

To locate only `lab_policy`:

```bash
aws iam list-policies \
    --scope Local \
    --query "Policies[?PolicyName=='lab_policy']"
```

This provides information such as:

* Policy ARN
* Policy ID
* Policy name
* Default version ID

---

## Step 2 — Retrieve the Policy Version

The policy ARN and default version ID were then used with:

```bash
aws iam get-policy-version \
    --policy-arn arn:aws:iam::<ACCOUNT-ID>:policy/lab_policy \
    --version-id v1
```

The command retrieves the IAM policy document associated with the specified policy version.

---

## Step 3 — Save the Policy Document

The policy information can be redirected to a JSON file using `>`:

```bash
aws iam get-policy-version \
    --policy-arn arn:aws:iam::<ACCOUNT-ID>:policy/lab_policy \
    --version-id v1 \
    > lab_policy.json
```

For a cleaner policy document containing only the `Document` section:

```bash
aws iam get-policy-version \
    --policy-arn arn:aws:iam::<ACCOUNT-ID>:policy/lab_policy \
    --version-id v1 \
    --query 'PolicyVersion.Document' \
    --output json \
    > lab_policy.json
```

The resulting file:

```text
lab_policy.json
```

contains the IAM policy in JSON format.

---

# Useful AWS CLI Commands

### Check AWS CLI version

```bash
aws --version
```

### Check current AWS CLI configuration

```bash
aws configure list
```

### Check AWS identity

```bash
aws sts get-caller-identity
```

### List IAM users

```bash
aws iam list-users
```

### List customer-managed policies

```bash
aws iam list-policies --scope Local
```

### Retrieve a specific policy version

```bash
aws iam get-policy-version \
    --policy-arn <POLICY-ARN> \
    --version-id <VERSION-ID>
```

---

# Security Best Practices

This lab involved AWS credentials and IAM permissions. The following security practices should always be followed:

* Never commit AWS access keys to GitHub.
* Never commit secret access keys.
* Never commit `.pem` private keys.
* Never publish passwords or temporary credentials.
* Use IAM policies based on **least privilege**.
* Remove temporary credentials after completing a lab.
* Use IAM roles instead of long-term access keys whenever possible.
* Use `.gitignore` to prevent sensitive files from being committed.

Example `.gitignore`:

```gitignore
# AWS credentials
.aws/
credentials
config

# SSH private keys
*.pem
*.key

# Environment files
.env
.env.*

# Lab credentials
secrets/
credentials.txt
```

---

# Key Takeaways

This lab demonstrated how to:

1. Install AWS CLI v2 on a Red Hat Linux EC2 instance.
2. Verify the AWS CLI installation.
3. Configure the AWS CLI to communicate with an AWS account.
4. Use AWS CLI commands to interact with IAM.
5. List IAM users from the command line.
6. Identify customer-managed IAM policies.
7. Retrieve an IAM policy version.
8. Export an IAM policy document to a JSON file.
9. Use AWS documentation and CLI Command Reference to solve an AWS challenge.
10. Apply basic AWS credential and IAM security practices.

---

# Skills Demonstrated

**AWS:**
EC2 · IAM · AWS CLI · IAM Policies · Policy Versions · Access Management

**Linux:**
SSH · Bash · curl · unzip · file redirection · command-line administration

**Security:**
IAM · Least Privilege · Access Keys · Credential Protection · Permissions

**Automation & CLI:**
AWS CLI · JSON · AWS API interaction · CLI-based troubleshooting

---

## Lab Outcome

Successfully installed and configured AWS CLI on a Red Hat Linux instance, established connectivity with an AWS account, queried IAM resources, and retrieved a customer-managed IAM policy using AWS CLI.

This lab demonstrates practical experience with **AWS command-line administration, IAM permissions, Linux environments, and security-focused AWS operations**.

