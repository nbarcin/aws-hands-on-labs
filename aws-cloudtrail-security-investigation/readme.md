# AWS CloudTrail Security Investigation & Incident Response

A hands-on AWS security investigation lab focused on detecting, investigating, and remediating a simulated web server compromise.

The project demonstrates how **AWS CloudTrail, Amazon S3, AWS CLI, Linux tools, Amazon Athena, IAM, Amazon EC2, and Security Groups** can be used together to investigate suspicious activity and improve the security posture of an AWS environment.

---

## Project Overview

A simulated Café website hosted on an Amazon EC2 instance was compromised.

During the investigation, an unauthorized user:

* Accessed the web server through SSH.
* Created an operating-system user.
* Modified the SSH configuration.
* Added an inbound Security Group rule allowing SSH access from the entire internet (`0.0.0.0/0`).
* Modified a website image.
* Used programmatic AWS access to modify the EC2 Security Group.

The objective was to determine:

1. Who performed the attack?
2. When did the attack occur?
3. What IP address was used?
4. How was the AWS environment accessed?
5. How did the attacker gain access to the EC2 instance?
6. How could the environment be secured?

---

## Architecture

```text
                    AWS Account
                        |
                        |
                 AWS CloudTrail
                        |
                        v
                  Amazon S3
              CloudTrail Log Bucket
                        |
             +----------+----------+
             |                     |
             v                     v
        AWS CLI / Linux        Amazon Athena
        grep / zcat            SQL Queries
             |                     |
             +----------+----------+
                        |
                        v
                Security Investigation
                        |
        +---------------+---------------+
        |                               |
        v                               v
   Amazon EC2                     IAM User
   Web Server                    Investigation
        |
        +----------------------+
        |
        v
  Security Group
  SSH / HTTP Rules
```

---

# AWS Services & Technologies

* **Amazon EC2** — Web server hosting the Café website
* **Amazon S3** — CloudTrail log storage
* **AWS CloudTrail** — API activity and auditing
* **Amazon Athena** — SQL-based CloudTrail log analysis
* **AWS IAM** — Identity and access management
* **AWS CLI** — Command-line investigation
* **Linux** — Server investigation and remediation
* **Security Groups** — Network access control
* **SSH** — Secure remote administration
* **Python** — JSON log formatting
* **SQL** — CloudTrail investigation queries

---

# Investigation Workflow

## 1. Identify the Web Server

The Café Web Server was located in Amazon EC2.

The initial Security Group contained an HTTP rule for port 80.

A controlled SSH rule was added for administration:

```text
Protocol: TCP
Port: 22
Source: My IP /32
```

The `/32` restriction limits SSH access to a single public IP address.

---

# 2. Detect the Compromise

After CloudTrail was configured, the Café website was refreshed.

The website showed evidence of unauthorized modification.

The EC2 Security Group was also inspected.

An unexpected inbound rule appeared:

```text
Type: SSH
Port: 22
Source: 0.0.0.0/0
```

This rule allows SSH access from anywhere on the internet and represents a significant security risk.

---

# 3. Enable CloudTrail Logging

A CloudTrail trail named:

```text
monitor
```

was configured to deliver logs to an Amazon S3 bucket.

Example:

```text
s3://monitoring####/
```

CloudTrail records AWS API activity, including information such as:

* Identity
* Event time
* Event source
* Event name
* Source IP address
* Request parameters
* AWS Region

---

# 4. Download CloudTrail Logs

The CloudTrail logs were copied from Amazon S3 to the EC2 instance for local analysis.

Example:

```bash
mkdir ctraillogs
cd ctraillogs

aws s3 ls

aws s3 cp s3://<monitoring-bucket>/ . --recursive
```

The logs were stored under a structure similar to:

```text
AWSLogs/
└── <account-id>/
    └── CloudTrail/
        └── us-west-2/
            └── 2026/
                └── 10/
                    └── 06/
```

CloudTrail log files were compressed as:

```text
.json.gz
```

The logs were inspected using:

```bash
zcat <filename>.json.gz
```

---

# 5. Analyze Logs with Linux

CloudTrail events are stored in JSON format.

A single log file can be formatted for easier inspection with:

```bash
cat <filename.json> | python3 -m json.tool
```

Important CloudTrail fields include:

```text
awsRegion
eventName
eventSource
eventTime
requestParameters
sourceIPAddress
userIdentity
```

---

## Searching CloudTrail Logs with grep

CloudTrail events were searched using Linux `grep`.

For example:

```bash
for i in $(ls); do
    echo $i
    cat $i | python3 -m json.tool | grep eventName
done
```

This makes it possible to quickly identify API operations recorded in the logs.

The investigation focused particularly on:

```text
Security Group
EC2
SSH
AuthorizeSecurityGroupIngress
```

---

# 6. Investigate with AWS CLI

CloudTrail also provides the `lookup-events` command.

Example:

```bash
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=ConsoleLogin
```

Security Group activity was investigated using:

```bash
aws cloudtrail lookup-events \
  --lookup-attributes \
  AttributeKey=ResourceType,AttributeValue=AWS::EC2::SecurityGroup \
  --output text
```

---

## Identify the Web Server Security Group

The EC2 instance's Region was retrieved using instance metadata:

```bash
region=$(curl http://169.254.169.254/latest/dynamic/instance-identity/document \
| grep region | cut -d '"' -f4)
```

The Security Group ID was then identified:

```bash
sgId=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values='Cafe Web Server'" \
  --query 'Reservations[*].Instances[*].SecurityGroups[*].[GroupId]' \
  --region $region \
  --output text)
```

The Security Group ID was displayed with:

```bash
echo $sgId
```

CloudTrail activity was then filtered using the Security Group ID:

```bash
aws cloudtrail lookup-events \
  --lookup-attributes \
  AttributeKey=ResourceType,AttributeValue=AWS::EC2::SecurityGroup \
  --region $region \
  --output text | grep $sgId
```

This helped narrow the investigation to events affecting the web server's Security Group.

---

# 7. Analyze CloudTrail Logs with Amazon Athena

Amazon Athena was used to query the CloudTrail data stored in Amazon S3 using SQL.

An Athena table was created from the CloudTrail S3 location.

Example table:

```text
cloudtrail_logs_monitoring####
```

The table maps CloudTrail JSON fields into Athena columns.

Important columns include:

```text
useridentity
eventtime
eventsource
eventname
sourceipaddress
requestparameters
```

The `useridentity` field is a nested structure, so nested fields can be accessed using dot notation.

---

## Basic Athena Query

```sql
SELECT *
FROM cloudtrail_logs_monitoring####
LIMIT 5;
```

---

## Investigate User Activity

```sql
SELECT
    useridentity.username,
    eventtime,
    eventsource,
    eventname,
    requestparameters
FROM cloudtrail_logs_monitoring####
LIMIT 30;
```

---

## Focus on EC2 Activity

```sql
SELECT
    useridentity.username,
    eventtime,
    eventsource,
    eventname,
    sourceipaddress,
    requestparameters
FROM cloudtrail_logs_monitoring####
WHERE eventsource = 'ec2.amazonaws.com'
ORDER BY eventtime;
```

---

## Search for Security Group Activity

```sql
SELECT
    useridentity.username,
    eventtime,
    eventname,
    sourceipaddress,
    requestparameters
FROM cloudtrail_logs_monitoring####
WHERE eventsource = 'ec2.amazonaws.com'
  AND eventname LIKE '%SecurityGroup%'
ORDER BY eventtime;
```

This query helps identify suspicious Security Group modifications.

---

## Recent CloudTrail Activity

A broader investigation query can also be used:

```sql
SELECT DISTINCT
    useridentity.username,
    eventname,
    eventsource
FROM cloudtrail_logs_monitoring####
WHERE from_iso8601_timestamp(eventtime) >
      date_add('day', -1, now())
ORDER BY eventsource;
```

---

# 8. Incident Findings

The investigation was used to determine:

| Investigation Item     | Finding                                  |
| ---------------------- | ---------------------------------------- |
| Compromised resource   | Café Web Server EC2 instance             |
| AWS service affected   | Amazon EC2                               |
| Security weakness      | SSH exposed to the internet              |
| Dangerous rule         | TCP/22 from `0.0.0.0/0`                  |
| Evidence source        | AWS CloudTrail                           |
| Log storage            | Amazon S3                                |
| Log analysis           | AWS CLI, Linux, Amazon Athena            |
| Identity investigation | IAM + CloudTrail                         |
| Network evidence       | Source IP address                        |
| Attack method          | Determined from CloudTrail event details |

> The actual attacker username, timestamp, and source IP are intentionally not hard-coded in this public README. They were identified during the hands-on investigation.

---

# 9. Operating System Investigation

The EC2 instance was also investigated at the Linux operating-system level.

Recently authenticated users were reviewed using:

```bash
sudo aureport --auth
```

Currently logged-in users were checked with:

```bash
who
```

A suspicious OS account named:

```text
chaos-user
```

was identified.

---

# 10. Remove the Suspicious OS User

The suspicious user's active session was terminated before deleting the account.

The process associated with the active session was stopped:

```bash
sudo kill -9 <process-number>
```

The active sessions were verified:

```bash
who
```

The suspicious user was then removed:

```bash
sudo userdel -r chaos-user
```

Other login-capable accounts were reviewed:

```bash
sudo cat /etc/passwd | grep -v nologin
```

---

# 11. Secure SSH Configuration

The SSH configuration was inspected:

```bash
sudo ls -l /etc/ssh/sshd_config
```

The configuration was edited:

```bash
sudo vi /etc/ssh/sshd_config
```

Password authentication was disabled:

```text
#PasswordAuthentication yes
PasswordAuthentication no
```

The SSH service was restarted:

```bash
sudo service sshd restart
```

This helps ensure that SSH authentication uses the configured key-based authentication mechanism rather than allowing password-based remote access.

---

# 12. Remove the Dangerous Security Group Rule

The unauthorized rule allowing SSH from the entire internet was removed.

The final intended SSH configuration was:

```text
SSH
TCP
22
My IP /32
```

rather than:

```text
SSH
TCP
22
0.0.0.0/0
```

This significantly reduces the exposed attack surface.

---

# 13. Restore the Compromised Website

The website image directory was inspected:

```bash
cd /var/www/html/cafe/images/
ls -l
```

A backup of the original image was identified.

The original image was restored:

```bash
sudo mv Coffee-and-Pastries.backup Coffee-and-Pastries.jpg
```

The website was then refreshed and verified.

---

# 14. Remove the Compromised IAM User

The suspicious AWS IAM user was removed from the account through the IAM console.

This completed the identity remediation at the AWS account level.

---

# Security Improvements

The following controls were implemented or reviewed:

### Network Security

* Removed SSH access from `0.0.0.0/0`
* Restricted SSH to the administrator's IP address
* Reviewed EC2 Security Group rules

### Identity & Access Management

* Identified the suspicious IAM user
* Removed the compromised IAM user
* Investigated AWS activity through CloudTrail
* Reviewed operating-system accounts

### Server Security

* Removed the unauthorized OS account
* Terminated the attacker's active session
* Disabled SSH password authentication
* Restarted the SSH service
* Restored the compromised website file

### Monitoring & Auditing

* Enabled AWS CloudTrail
* Stored logs in Amazon S3
* Investigated logs using AWS CLI
* Used Linux tools for local log analysis
* Used Amazon Athena for SQL-based investigation

---

# Key Security Lessons

## 1. CloudTrail is critical for incident investigation

Without CloudTrail, determining who performed an AWS API operation can be significantly more difficult.

CloudTrail provides valuable evidence such as:

```text
Who?
When?
What?
Where from?
Which AWS service?
Which API operation?
What parameters?
```

---

## 2. Never expose SSH unnecessarily

Avoid:

```text
0.0.0.0/0 → TCP 22
```

Prefer:

```text
Administrator IP → TCP 22
```

or, for production environments, consider stronger administrative access patterns such as AWS Systems Manager Session Manager.

---

## 3. AWS security and OS security are connected

Securing IAM alone is not enough.

The investigation demonstrated two layers of compromise:

```text
AWS Account
     |
     v
IAM / AWS API
     |
     v
EC2 Security Group
     |
     v
EC2 Instance
     |
     v
Linux OS User
     |
     v
Website Files
```

Security must therefore be addressed across both the AWS control plane and the operating system.

---

# Skills Demonstrated

```text
AWS CloudTrail
Amazon S3
Amazon Athena
Amazon EC2
AWS IAM
AWS CLI
Security Groups
Linux
SSH
Bash
grep
JSON
Python
SQL
Incident Response
Security Investigation
Log Analysis
Access Control
Network Security
```


# Example Investigation Flow

```text
Suspicious Website
       |
       v
Check EC2
       |
       v
Check Security Group
       |
       v
SSH 22 from 0.0.0.0/0
       |
       v
Enable / Analyze CloudTrail
       |
       +----------------------+
       |                      |
       v                      v
AWS CLI / grep           Amazon Athena
       |                      |
       +----------+-----------+
                  |
                  v
          Identify AWS User
          Event Time
          Source IP
          API Operation
                  |
                  v
          Investigate EC2
                  |
                  v
          Investigate Linux
                  |
                  v
        Remove Unauthorized User
                  |
                  v
        Harden SSH Configuration
                  |
                  v
        Remove Security Group Rule
                  |
                  v
        Restore Website
                  |
                  v
        Remove Compromised IAM User
```

---

# Outcome

This project demonstrates a complete AWS security investigation workflow from **detection to remediation**.

The investigation used AWS-native services and Linux tools to correlate:

```text
AWS API activity
        +
Network access
        +
EC2 configuration
        +
Linux authentication
        +
Website file changes
```

The environment was subsequently hardened by removing unauthorized access, restricting SSH, disabling password authentication, removing the compromised IAM/OS users, and restoring the affected website content.

