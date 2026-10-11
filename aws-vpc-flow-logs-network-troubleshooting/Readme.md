# AWS VPC Flow Logs & Network Troubleshooting

## Project Overview

This hands-on AWS lab demonstrates how to troubleshoot network connectivity issues in an Amazon Virtual Private Cloud (VPC) using the AWS Command Line Interface (AWS CLI) and VPC Flow Logs.

The lab focuses on identifying and resolving network configuration issues affecting an EC2 web server, capturing network traffic, and analyzing rejected connections to understand how AWS network security controls work.

## Objectives

* Configure and use the AWS CLI from an EC2 CLI Host instance.
* Create an Amazon S3 bucket for VPC Flow Logs.
* Enable VPC Flow Logs to capture accepted and rejected IP traffic.
* Troubleshoot EC2 connectivity issues using AWS CLI commands.
* Inspect Security Groups, Route Tables, Internet Gateways, and Network ACLs.
* Identify network traffic rejected on SSH port 22.
* Download, decompress, and analyze flow log files using Linux utilities.
* Verify network connectivity after resolving configuration issues.

## AWS Services & Tools

* **Amazon VPC** — Virtual networking and subnet configuration
* **Amazon EC2** — CLI Host and web server instances
* **AWS CLI** — Programmatic AWS resource inspection and management
* **Amazon S3** — Storage for VPC Flow Logs
* **VPC Flow Logs** — Network traffic monitoring and troubleshooting
* **Security Groups** — Instance-level stateful traffic filtering
* **Network ACLs** — Subnet-level stateless traffic filtering
* **Route Tables & Internet Gateways** — Network routing and internet connectivity
* **Linux CLI** — `grep`, `awk`, `head`, `gunzip`, `date`, and `nmap`

## Lab Architecture

The lab uses a VPC containing a public subnet and an EC2 web server, along with a CLI Host instance used to investigate AWS resources through the command line.

VPC Flow Logs capture traffic information and deliver log files to an S3 bucket for analysis.

**High-level workflow:**

1. Connect to the CLI Host instance.
2. Configure AWS CLI access.
3. Create an S3 bucket and enable VPC Flow Logs.
4. Investigate web server connectivity failures.
5. Correct routing and network ACL configuration issues.
6. Download and analyze flow log records.
7. Verify the results.

## Implementation Steps

### 1. Configure AWS CLI

Connect to the CLI Host instance using EC2 Instance Connect and verify the CLI configuration.

```bash
aws --version
aws sts get-caller-identity
aws configure get region
```

The lab uses the `us-west-2` (Oregon) Region.

For production environments, prefer an appropriately scoped EC2 IAM role and temporary credentials instead of long-lived access keys.

### 2. Create an S3 Bucket for Flow Logs

Create a globally unique S3 bucket name and use it as the destination for network traffic logs.

```bash
aws s3api create-bucket \
  --bucket YOUR-UNIQUE-FLOW-LOG-BUCKET \
  --region us-west-2 \
  --create-bucket-configuration LocationConstraint=us-west-2
```

### 3. Enable VPC Flow Logs

Identify the VPC ID:

```bash
aws ec2 describe-vpcs \
  --filters "Name=tag:Name,Values=VPC1" \
  --query 'Vpcs[*].[VpcId,Tags, CidrBlock]' \
  --output table
```

Create a flow log that captures all traffic:

```bash
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids YOUR-VPC-ID \
  --traffic-type ALL \
  --log-destination-type s3 \
  --log-destination arn:aws:s3:::YOUR-UNIQUE-FLOW-LOG-BUCKET
```

Verify the flow log status:

```bash
aws ec2 describe-flow-logs \
  --query 'FlowLogs[*].[FlowLogId,ResourceId,FlowLogStatus,LogDestination]' \
  --output table
```

### 4. Troubleshoot EC2 Connectivity

Inspect the web server instance:

```bash
aws ec2 describe-instances \
  --filters "Name=ip-address,Values=YOUR-WEB-SERVER-PUBLIC-IP" \
  --query 'Reservations[*].Instances[*].[State.Name,PrivateIpAddress,InstanceId,SubnetId,SecurityGroups]' \
  --output json
```

Investigate the following components:

* **Security Groups:** Verify inbound HTTP (port 80) and SSH (port 22) rules as required by the lab.
* **Route Tables:** Confirm that the public subnet has a default route (`0.0.0.0/0`) to the correct Internet Gateway.
* **Network ACLs:** Inspect inbound and outbound rules, including the required return traffic and ephemeral ports.
* **Network Interfaces:** Identify the network interface associated with the web server's public IP address.

Use Nmap, if available and authorized in the lab environment, to inspect reachable ports.

```bash
nmap YOUR-WEB-SERVER-PUBLIC-IP
```

Apply only the configuration changes required to resolve the identified issues.

### 5. Download and Analyze Flow Logs

Download the flow log files from S3:

```bash
mkdir -p ~/flowlogs
cd ~/flowlogs

aws s3 cp \
  s3://YOUR-UNIQUE-FLOW-LOG-BUCKET/ \
  . \
  --recursive
```

Locate and decompress downloaded log files:

```bash
find . -type f -name '*.log.gz' -exec gunzip {} \;
```

Inspect a log file:

```bash
head -n 5 YOUR-FLOW-LOG-FILE.log
```

Count rejected records:

```bash
grep -h 'REJECT' ./*.log | wc -l
```

For the default VPC Flow Logs record format, filter rejected traffic destined for SSH port 22:

```bash
awk '$7 == 22 && $13 == "REJECT"' ./*.log
```

The default format places the destination port in field 7 and the action in field 13. Check the configured record format before using positional filters.

### 6. Convert Unix Timestamps

Convert a flow log timestamp into a readable date:

```bash
date -d @UNIX_TIMESTAMP
```

This helps correlate network events with troubleshooting attempts.

## Key Learnings

* How to configure and use AWS CLI for network troubleshooting.
* How VPC Flow Logs capture accepted and rejected traffic.
* How Security Groups and Network ACLs differ in traffic filtering.
* How routing and Internet Gateway configuration affect public subnet connectivity.
* How to investigate SSH and HTTP connectivity failures.
* How to use Linux command-line utilities to analyze network logs.
* How to correlate network traffic records with EC2 network interfaces.

## Security Considerations

* Follow the principle of least privilege when assigning IAM permissions.
* Prefer temporary credentials and IAM roles over long-lived access keys.
* Do not commit AWS credentials, account-specific secrets, or sensitive log data to GitHub.
* Restrict SSH access to trusted source IP addresses when required.
* Avoid broad network rule changes; validate each change against the intended architecture.
* Delete lab resources when they are no longer needed to prevent unnecessary charges.



**Project Type:** AWS Networking & Security Hands-On Lab
**Services:** Amazon VPC, EC2, S3, AWS CLI, VPC Flow Logs
**Focus Areas:** Network Troubleshooting, Traffic Monitoring, Log Analysis, Cloud Security

