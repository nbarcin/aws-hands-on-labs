# AWS Systems Manager Hands-On Lab

## Overview

This hands-on lab explores key capabilities of **AWS Systems Manager (SSM)** for managing EC2 instances securely and efficiently without relying on direct SSH access.

The lab covers inventory collection, remote command execution, centralized application configuration, and secure interactive sessions using a sample web application called **Widget Manufacturing Dashboard**.

## Objectives

By completing this lab, you will learn how to:

* Collect software and configuration inventory from managed EC2 instances.
* Install a custom application using Systems Manager Run Command.
* Manage application settings using Parameter Store.
* Access an EC2 instance securely through Session Manager.
* Understand the roles of IAM permissions, SSM Agent, and network connectivity.

## AWS Services Used

* Amazon EC2
* AWS Systems Manager
* Fleet Manager and Inventory
* Run Command
* Parameter Store
* Session Manager
* AWS Identity and Access Management (IAM)
* AWS CloudTrail

## Prerequisites

Before starting, ensure that:

* You have access to an AWS account or training sandbox.
* A running EC2 instance named `Managed Instance` is available.
* SSM Agent is installed and running.
* The instance has an appropriate IAM role, commonly based on `AmazonSSMManagedInstanceCore`.
* The instance can communicate with the required Systems Manager endpoints.
* The training environment provides the custom document `Install Dashboard App`.
* The instance has the appropriate network configuration for accessing the sample web application.

> **Note:** AWS Console navigation may change over time. Use the current Systems Manager console and official AWS documentation if a menu label differs from the instructions below.

---

## Task 1: Generate Inventory Lists for Managed Instances

### Objective

Collect operating system information, installed applications, and configuration metadata from a managed EC2 instance.

### Steps

1. Sign in to the AWS Management Console.
2. Search for **Systems Manager** and open the service.
3. Navigate to **Node Management → Inventory**. Depending on the console layout, inventory setup may also be accessible through Fleet Manager.
4. Choose **Setup Inventory**, if available.
5. Select **Manually selecting instances** as the target method.
6. Select `Managed Instance`.
7. If the setup form includes an association name, enter `Inventory-Association`.
8. Leave other settings at their defaults unless the lab specifies otherwise.
9. Choose **Setup Inventory** to create the association.
10. Open **State Manager** and verify that the inventory association was created.
11. Wait for the association to run successfully.
12. Return to the managed node details and open the **Inventory** tab.
13. Review the available inventory types and installed applications.

### Expected Result

An Inventory association is configured, and software inventory data becomes available after collection completes.

### Key Learning

Systems Manager Inventory helps administrators review and validate software configurations without manually connecting to every instance through SSH.

---

## Task 2: Install a Custom Application Using Run Command

### Objective

Install the Widget Manufacturing Dashboard on an EC2 instance without directly connecting to the instance through SSH.

### Steps

1. In Systems Manager, navigate to **Node Management → Run Command**.
2. Choose **Run command**.
3. Search for the custom document `Install Dashboard App`.
4. Select the document provided by the training environment.
5. Verify the document version. Use the default version specified by the lab.
6. Under **Targets**, select **Choose instances manually**.
7. Select `Managed Instance`.
8. Under **Output options**, leave **Enable an S3 bucket** disabled if required by the lab.
9. Review the remaining settings.
10. Choose **Run**.
11. Note the generated **Command ID**.
12. Monitor the command status until it becomes `Success`.
13. If using Vocareum, open **Details → Show** and copy the `ServerIP` value.
14. Open the following address in a browser, replacing the placeholder with the actual IP:

`http://<ServerIP>`

15. Verify that the Widget Manufacturing Dashboard loads.

### What the Installation Does

According to the lab instructions, the installation document runs a script that installs:

* Apache web server
* PHP
* AWS SDK
* Widget Manufacturing Dashboard

The script also starts the web server.

### Expected Result

The custom application is installed successfully and can be accessed using the lab-provided address.

### Troubleshooting

* `Install Dashboard App` is a custom training document and may not exist in a regular AWS account.
* If the document is missing, verify that you are using the correct training environment.
* If the command fails, inspect the command output and verify SSM Agent status and IAM permissions.
* If the website does not load, check the public IP address, web server status, and security group rules.
* Allow HTTP access only from the sources needed for the lab.

---

## Task 3: Use Parameter Store to Manage Application Settings

### Objective

Store an application feature flag in Parameter Store and observe how the dashboard responds to the setting.

### Steps

1. Keep the dashboard browser tab open.
2. Return to the Systems Manager console.
3. Navigate to **Application Management → Parameter Store**.
4. Choose **Create parameter**.
5. Configure the parameter as follows:

| Setting     | Value                           |
| ----------- | ------------------------------- |
| Name        | `/dashboard/show-beta-features` |
| Description | `Display beta features`         |
| Tier        | Standard                        |
| Type        | String                          |
| Value       | `True`                          |

6. Choose **Create parameter**.
7. Confirm that the parameter was created successfully.
8. Return to the dashboard browser tab.
9. Refresh the page.
10. Verify whether the additional beta chart appears.

### Optional Test

1. Return to Parameter Store.
2. Delete `/dashboard/show-beta-features` if you want to test disabling the feature and the lab permits deletion.
3. Refresh the dashboard.
4. Verify whether the beta chart disappears.

### Expected Result

The application uses a centrally stored parameter to control whether the beta feature is displayed.

### Key Learning

Parameter Store provides centralized storage for application configuration values.

The application must be programmed to read the parameter, and its IAM permissions must allow the required access. Creating a parameter alone does not automatically change application behavior.

For sensitive information, use `SecureString` with appropriate IAM and AWS KMS permissions.

---

## Task 4: Access EC2 Instances Using Session Manager

### Objective

Open an interactive shell on the managed EC2 instance without using SSH or opening inbound port 22.

### Steps

1. In Systems Manager, navigate to **Node Management → Session Manager**.
2. Choose **Start session**.
3. Select `Managed Instance`.
4. Choose **Start session** again.
5. A browser-based terminal opens.
6. Run the following command to list the application files:

```bash
ls /var/www/html
```

### Retrieve the AWS Region Using IMDSv2

The following commands retrieve the instance's Availability Zone using Instance Metadata Service Version 2 (IMDSv2) and derive the AWS Region.

```bash
# Request an IMDSv2 token
TOKEN=$(curl -sS -X PUT \
  "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

# Retrieve the Availability Zone
AZ=$(curl -sS \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/placement/availability-zone)

# Derive the AWS Region
export AWS_DEFAULT_REGION="${AZ%?}"
```

### List EC2 Instances

Run the following command:

```bash
aws ec2 describe-instances
```

This command returns EC2 instance information in JSON format, subject to the permissions of the AWS identity used by the CLI.

**Important:** The instance role must have the `ec2:DescribeInstances` permission for this command. Systems Manager access does not automatically grant permission to call EC2 APIs.

### Expected Result

You can inspect application files and execute authorized shell commands through Session Manager without using SSH.

### Security Benefits

Session Manager helps reduce the need to:

* Open inbound SSH port 22.
* Maintain bastion hosts.
* Manage SSH key pairs for interactive access.

Access can be controlled through IAM. Session activity logging can also be configured through supported Amazon CloudWatch Logs or Amazon S3 options.

---

## Summary

In this lab, I practiced four important AWS Systems Manager capabilities:

| Task | Feature         | Purpose                                          |
| ---- | --------------- | ------------------------------------------------ |
| 1    | Inventory       | Collect software and configuration metadata      |
| 2    | Run Command     | Execute remote commands and install applications |
| 3    | Parameter Store | Manage application configuration centrally       |
| 4    | Session Manager | Access an EC2 instance without SSH               |

## Key Takeaways

* Systems Manager can simplify centralized EC2 management.
* Inventory helps track installed software and configuration metadata.
* Run Command executes commands on managed nodes without requiring an interactive SSH connection.
* Parameter Store provides centralized configuration management.
* Session Manager supports secure, auditable instance access.
* Correct IAM permissions, SSM Agent status, and network connectivity are essential for successful operation.


