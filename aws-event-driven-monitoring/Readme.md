# AWS CloudWatch Monitoring, Alerting & Compliance Lab

## Overview

This hands-on AWS lab demonstrates how to monitor an EC2 web server using Amazon CloudWatch, AWS Systems Manager, Amazon SNS, Amazon EventBridge, and AWS Config.

The lab focuses on practical AWS monitoring, logging, alerting, event-driven notifications, and infrastructure compliance.

During this lab, I:

* Installed the CloudWatch Agent on an EC2 instance
* Used AWS Systems Manager Run Command to manage the agent
* Stored the CloudWatch Agent configuration in Systems Manager Parameter Store
* Collected Apache web server logs
* Collected CPU, memory, disk, disk I/O, and swap metrics
* Sent application logs to CloudWatch Logs
* Created a CloudWatch Logs metric filter for HTTP 404 errors
* Created a CloudWatch alarm based on application log data
* Configured Amazon SNS email notifications
* Monitored EC2 instance metrics
* Created an Amazon EventBridge rule for EC2 state changes
* Sent real-time EC2 notifications through SNS
* Created AWS Config compliance rules
* Checked AWS resource compliance

---

# Architecture

```text
                           AWS Cloud
                              |
                       +------+------+
                       |             |
                    EC2 Web       Systems
                     Server       Manager
                       |             |
                       |       Run Command
                       |             |
                       +----- CloudWatch Agent
                                  |
                         +--------+--------+
                         |                 |
                       Logs              Metrics
                         |                 |
                         v                 v
                  CloudWatch Logs   CloudWatch Metrics
                         |                 |
                         v                 v
                  Metric Filter       CloudWatch Alarm
                         |                 |
                         |                 v
                         +--------------> SNS
                                            |
                                            v
                                          Email


                 EC2 State Change
                        |
                        v
                   EventBridge
                        |
                        v
                       SNS
                        |
                        v
                      Email


                  AWS Resources
                        |
                        v
                    AWS Config
                        |
                        v
               Compliance Rules
                        |
                +-------+-------+
                |               |
            Compliant       Noncompliant
```

---

# AWS Services Used

| Service             | Purpose                                           |
| ------------------- | ------------------------------------------------- |
| Amazon EC2          | Web server monitored in the lab                   |
| Amazon CloudWatch   | Metrics, logs, alarms, and monitoring             |
| CloudWatch Agent    | Collect system-level metrics and application logs |
| AWS Systems Manager | Install and configure the CloudWatch Agent        |
| Parameter Store     | Store the CloudWatch Agent configuration          |
| CloudWatch Logs     | Centralized application log collection            |
| CloudWatch Metrics  | Store and visualize monitoring metrics            |
| CloudWatch Alarms   | Trigger actions when thresholds are reached       |
| Amazon SNS          | Send email notifications                          |
| Amazon EventBridge  | Detect EC2 state changes                          |
| AWS Config          | Evaluate AWS resource compliance                  |

---

# Prerequisites

Before starting this lab, I used:

* An AWS account
* An EC2 Linux instance
* An Apache web server
* An IAM role attached to the EC2 instance
* AWS Systems Manager access
* A verified email address for SNS notifications
* Access to the AWS Management Console

The EC2 instance used throughout the lab is referred to as:

```text
Web Server
```

---

# Task 1 — Install the CloudWatch Agent

## Objective

The standard EC2 CloudWatch metrics provide information such as CPU utilization and network activity.

However, I wanted to monitor information from inside the operating system, including:

* Memory utilization
* Disk utilization
* Disk I/O
* Swap utilization
* Apache application logs

For this purpose, I installed the Amazon CloudWatch Agent.

---

## Step 1 — Open AWS Systems Manager

From the AWS Management Console:

```text
AWS Console
    ↓
Systems Manager
    ↓
Run Command
```

I selected:

```text
Run a command
```

Then I searched for:

```text
AWS-ConfigureAWSPackage
```

---

## Step 2 — Install AmazonCloudWatchAgent

I configured the command with:

```text
Action: Install
Name: AmazonCloudWatchAgent
Version: latest
```

Under Targets, I selected:

```text
Choose instances manually
```

Then selected:

```text
Web Server
```

Finally, I selected:

```text
Run
```

I waited until the overall status changed to:

```text
Success
```

The output confirmed that the CloudWatch Agent was successfully installed.

---

# Task 1.1 — Create the CloudWatch Agent Configuration

After installing the agent, I needed to tell it what information to collect.

Instead of storing the configuration directly on the EC2 instance, I used:

```text
AWS Systems Manager Parameter Store
```

This allows the CloudWatch Agent to retrieve the configuration from a centralized AWS service.

---

## Step 1 — Create a Parameter

I navigated to:

```text
Systems Manager
    ↓
Parameter Store
    ↓
Create parameter
```

I created:

```text
Name:
Monitor-Web-Server

Description:
Collect web logs and system metrics
```

---

## Step 2 — CloudWatch Agent Configuration

I used the following configuration:

```json
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "log_group_name": "HttpAccessLog",
            "file_path": "/var/log/httpd/access_log",
            "log_stream_name": "{instance_id}",
            "timestamp_format": "%b %d %H:%M:%S"
          },
          {
            "log_group_name": "HttpErrorLog",
            "file_path": "/var/log/httpd/error_log",
            "log_stream_name": "{instance_id}",
            "timestamp_format": "%b %d %H:%M:%S"
          }
        ]
      }
    }
  },
  "metrics": {
    "metrics_collected": {
      "cpu": {
        "measurement": [
          "cpu_usage_idle",
          "cpu_usage_iowait",
          "cpu_usage_user",
          "cpu_usage_system"
        ],
        "metrics_collection_interval": 10,
        "totalcpu": false
      },
      "disk": {
        "measurement": [
          "used_percent",
          "inodes_free"
        ],
        "metrics_collection_interval": 10,
        "resources": [
          "*"
        ]
      },
      "diskio": {
        "measurement": [
          "io_time"
        ],
        "metrics_collection_interval": 10,
        "resources": [
          "*"
        ]
      },
      "mem": {
        "measurement": [
          "mem_used_percent"
        ],
        "metrics_collection_interval": 10
      },
      "swap": {
        "measurement": [
          "swap_used_percent"
        ],
        "metrics_collection_interval": 10
      }
    }
  }
}
```

The configuration collects two categories of information.

### Application Logs

```text
/var/log/httpd/access_log
/var/log/httpd/error_log
```

### System Metrics

```text
CPU
Disk
Disk I/O
Memory
Swap
```

---

# Task 1.2 — Configure and Start the CloudWatch Agent

I returned to:

```text
Systems Manager
    ↓
Run Command
```

I searched for:

```text
AmazonCloudWatch-ManageAgent
```

I configured:

```text
Action: configure
Mode: ec2
Configuration Source: ssm
Configuration Location: Monitor-Web-Server
Restart: yes
```

I selected:

```text
Web Server
```

and ran the command.

After the command completed successfully, the CloudWatch Agent started using the configuration stored in Parameter Store.

---

# What I Learned from Task 1

This task helped me understand the difference between standard CloudWatch monitoring and agent-based monitoring.

Without the CloudWatch Agent, EC2 provides standard infrastructure metrics.

With the CloudWatch Agent, I can collect additional information from inside the operating system.

```text
EC2
 |
 +---- Standard CloudWatch Metrics
 |       |
 |       +-- CPU
 |       +-- Network
 |
 +---- CloudWatch Agent
         |
         +-- Memory
         +-- Disk
         +-- Disk I/O
         +-- Swap
         +-- Application Logs
```

I also learned how Systems Manager can be used to install and manage software on EC2 instances without manually connecting to the server.

---

# Task 2 — Monitor Application Logs with CloudWatch Logs

## Objective

The next step was to verify that Apache logs were being sent from the EC2 instance to CloudWatch Logs.

The Apache web server generates:

```text
Access Logs
Error Logs
```

The CloudWatch Agent sends these logs to:

```text
Amazon CloudWatch Logs
```

---

# Step 2.1 — Generate a 404 Error

I opened the Web Server IP address:

```text
http://<WebServerIP>
```

The Apache test page appeared.

Then I requested a page that did not exist:

```text
http://<WebServerIP>/start
```

The server returned:

```text
404 Not Found
```

This generated a new entry in the Apache access log.

---

# Step 2.2 — View the Logs

I navigated to:

```text
CloudWatch
    ↓
Logs
    ↓
Log groups
```

I found:

```text
HttpAccessLog
HttpErrorLog
```

I opened:

```text
HttpAccessLog
```

and then opened the log stream associated with the EC2 instance.

I found the request for:

```text
/start
```

with:

```text
HTTP Status Code: 404
```

This confirmed that the CloudWatch Agent was successfully shipping Apache logs to CloudWatch Logs.

---

# What I Learned from Task 2

I learned that CloudWatch Logs can provide centralized access to application logs.

Instead of logging into every EC2 instance individually:

```text
Server 1 ──┐
Server 2 ──┼──> CloudWatch Logs
Server 3 ──┘
Server 4 ──┘
```

logs can be collected and analyzed centrally.

This becomes especially useful when an application has multiple EC2 instances or an Auto Scaling environment.

---

# Task 2.3 — Create a Metric Filter

## Objective

Next, I wanted CloudWatch to count HTTP 404 errors automatically.

I selected:

```text
HttpAccessLog
```

and created a metric filter.

I used:

```text
[ip, id, user, timestamp, request, status_code=404, size]
```

This filter identifies log entries where:

```text
status_code = 404
```

---

## Metric Configuration

I created:

```text
Filter Name:
404Errors

Metric Namespace:
LogMetrics

Metric Name:
404Errors

Metric Value:
1
```

The workflow became:

```text
Apache Access Log
       |
       v
CloudWatch Logs
       |
       v
Metric Filter
       |
       v
404Errors Metric
```

---

# What I Learned About Metric Filters

A CloudWatch Logs metric filter allows log information to become a numerical CloudWatch metric.

For example:

```text
404 log
404 log
404 log
404 log
404 log
```

becomes:

```text
404Errors = 5
```

This metric can then be used by a CloudWatch Alarm.

---

# Task 2.4 — Create a CloudWatch Alarm

I created an alarm using the `404Errors` metric.

The condition was:

```text
404Errors >= 5
```

with a:

```text
Period: 1 minute
```

I configured Amazon SNS to send the notification.

The workflow became:

```text
404 Errors
     |
     v
Metric Filter
     |
     v
CloudWatch Metric
     |
     v
CloudWatch Alarm
     |
     v
SNS
     |
     v
Email
```

---

# Task 2.5 — Test the Alarm

I generated multiple invalid requests:

```text
http://<WebServerIP>/start1
http://<WebServerIP>/start2
http://<WebServerIP>/start3
http://<WebServerIP>/start4
http://<WebServerIP>/start5
```

Each request generated a 404 log entry.

After CloudWatch processed the events, the alarm changed to:

```text
ALARM
```

I also received an SNS email notification.

---

# What I Learned from the Alarm

This demonstrated how application logs can be converted into actionable monitoring events.

The complete workflow is:

```text
Application
     |
     v
Log
     |
     v
CloudWatch Logs
     |
     v
Metric Filter
     |
     v
Metric
     |
     v
Alarm
     |
     v
SNS
     |
     v
Notification
```

This is a practical example of application monitoring and alerting.

---

# Task 3 — Monitor EC2 Metrics

## Objective

In this task, I compared standard EC2 metrics with metrics collected by the CloudWatch Agent.

---

# Step 3.1 — View Standard EC2 Metrics

I navigated to:

```text
EC2
    ↓
Instances
    ↓
Web Server
    ↓
Monitoring
```

I reviewed metrics such as:

* CPU utilization
* Network traffic
* Network packets
* Disk activity

These metrics are available through standard AWS monitoring.

---

# Step 3.2 — View CloudWatch Agent Metrics

I navigated to:

```text
CloudWatch
    ↓
Metrics
    ↓
All metrics
    ↓
CWAgent
```

I explored metrics related to:

```text
CPU
Memory
Disk
Disk I/O
Swap
```

Examples included:

```text
mem_used_percent
disk_used_percent
swap_used_percent
io_time
```

---

# Standard Metrics vs Agent Metrics

An important concept I learned is:

```text
Standard EC2 Monitoring
        |
        +-- CPU
        +-- Network
        +-- Disk activity


CloudWatch Agent
        |
        +-- Memory
        +-- Disk space
        +-- Swap
        +-- Disk I/O
        +-- Application Logs
```

The CloudWatch Agent provides deeper visibility into the operating system.

---

# Task 4 — Real-Time EC2 Notifications with EventBridge

## Objective

I created an event-driven notification for EC2 state changes.

> Note: Older AWS training material refers to this functionality as **CloudWatch Events**. The current AWS service is **Amazon EventBridge**.

---

# Step 4.1 — Create an EventBridge Rule

I navigated to:

```text
Amazon EventBridge
    ↓
Rules
    ↓
Create rule
```

I created:

```text
Instance_Stopped_Terminated
```

The event source was:

```text
AWS Services
```

The AWS service was:

```text
EC2
```

The event type was:

```text
EC2 Instance State-change Notification
```

I selected these states:

```text
stopped
terminated
```

---

# Step 4.2 — Configure SNS as the Target

I configured:

```text
Target Type:
AWS Service

Target:
SNS Topic
```

The EventBridge rule sends the EC2 state-change event to SNS.

The architecture became:

```text
EC2
 |
 | State Change
 v
EventBridge
 |
 v
SNS
 |
 v
Email
```

---

# Step 4.3 — Test the Rule

I stopped the Web Server EC2 instance.

The instance changed from:

```text
Running
```

to:

```text
Stopping
```

and then:

```text
Stopped
```

EventBridge detected the state change and sent an SNS notification.

---

# What I Learned from Task 4

I learned how AWS services can work together in an event-driven architecture.

Instead of continuously checking the EC2 instance:

```text
Check EC2
Check EC2
Check EC2
```

EventBridge reacts when an event actually occurs.

```text
EC2 State Change
       |
       v
EventBridge
       |
       v
Target
```

This is an important concept for serverless and event-driven AWS architectures.

---

# Task 5 — Infrastructure Compliance with AWS Config

## Objective

In this task, I used AWS Config to evaluate AWS resource configurations.

AWS Config can be used for:

* Compliance
* Auditing
* Security analysis
* Configuration tracking
* Change management
* Troubleshooting

---

# Step 5.1 — Configure AWS Config

I opened:

```text
AWS Config
```

and completed the initial configuration when required.

Then I navigated to:

```text
AWS Config
    ↓
Rules
```

---

# Step 5.2 — Create the Required Tags Rule

I searched for the AWS managed rule:

```text
required-tags
```

I configured the required tag key:

```text
project
```

This rule checks whether AWS resources have the required tag.

The result can be:

```text
Compliant
```

or:

```text
Noncompliant
```

---

# Step 5.3 — Create the EBS Volume Rule

I added another AWS managed rule:

```text
ec2-volume-inuse-check
```

This rule checks whether EBS volumes are attached to EC2 instances.

After AWS Config evaluated the resources, I reviewed the compliance results.

---

# What I Learned from AWS Config

AWS Config is different from CloudWatch.

### CloudWatch

Focuses mainly on:

```text
Performance
Logs
Metrics
Alarms
Operational monitoring
```

### AWS Config

Focuses mainly on:

```text
Configuration
Compliance
Resource state
Configuration history
```

A simple way to remember:

```text
CloudWatch = "How is my system performing?"

AWS Config = "Is my resource configured correctly?"
```

---

# Complete End-to-End Workflow

The lab combined multiple AWS services into one monitoring architecture.

## Application Monitoring

```text
EC2 Web Server
      |
      v
CloudWatch Agent
      |
      v
Apache Logs
      |
      v
CloudWatch Logs
      |
      v
Metric Filter
      |
      v
404Errors Metric
      |
      v
CloudWatch Alarm
      |
      v
SNS
      |
      v
Email
```

## Infrastructure Event Monitoring

```text
EC2 State Change
      |
      v
EventBridge
      |
      v
SNS
      |
      v
Email
```

## Infrastructure Compliance

```text
AWS Resources
      |
      v
AWS Config
      |
      v
Config Rules
      |
      +------> Compliant
      |
      +------> Noncompliant
```

---

# Troubleshooting

## CloudWatch Log Group Does Not Appear

Possible causes:

* CloudWatch Agent is not running
* Incorrect log file path
* Missing IAM permissions
* Incorrect Parameter Store configuration
* Apache is not generating logs

I would check:

```text
/var/log/httpd/access_log
/var/log/httpd/error_log
```

and verify the CloudWatch Agent configuration.

---

## CloudWatch Agent Does Not Start

I would check:

1. EC2 IAM role
2. Systems Manager connectivity
3. Parameter Store parameter name
4. CloudWatch Agent configuration
5. Run Command output
6. CloudWatch Agent status

---

## Metric Filter Does Not Detect 404 Errors

I would verify that the web server actually generated:

```text
HTTP 404
```

For example:

```text
http://<WebServerIP>/test404
```

Then I would wait for the log event to reach CloudWatch Logs.

---

## Alarm Does Not Trigger

I would check the complete chain:

```text
Application
    ↓
Log
    ↓
CloudWatch Logs
    ↓
Metric Filter
    ↓
Metric
    ↓
Alarm
    ↓
SNS
    ↓
Email
```

I would also verify that the SNS email subscription has been confirmed.

---

## EventBridge Notification Does Not Arrive

I would check:

* EventBridge rule is enabled
* EC2 state-change event is configured correctly
* SNS topic is correct
* SNS subscription is confirmed
* EC2 actually changed to `stopped` or `terminated`

---

# Security Considerations

For a production environment, I would:

* Follow the principle of least privilege for IAM
* Avoid hard-coded credentials
* Protect sensitive log data
* Encrypt sensitive data
* Configure appropriate CloudWatch log retention
* Protect SNS topics
* Review AWS Config compliance regularly
* Avoid exposing AWS account information in public screenshots
* Never publish access keys or secrets
---

# Skills Demonstrated

This lab demonstrates hands-on experience with:

* AWS CloudWatch
* CloudWatch Logs
* CloudWatch Metrics
* CloudWatch Agent
* CloudWatch Alarms
* CloudWatch Metric Filters
* AWS Systems Manager
* Systems Manager Run Command
* Systems Manager Parameter Store
* Amazon EC2
* Amazon SNS
* Amazon EventBridge
* AWS Config
* Application Monitoring
* Infrastructure Monitoring
* Log Monitoring
* Alerting
* Event-Driven Architecture
* Infrastructure Compliance
* AWS Observability

---

# Key Concepts I Learned

## 1. CloudWatch Agent

The CloudWatch Agent runs inside an EC2 instance and can collect system-level metrics and application logs.

---

## 2. CloudWatch Logs

CloudWatch Logs provides centralized storage and analysis of logs from applications and infrastructure.

---

## 3. Metric Filters

Metric filters transform matching log events into CloudWatch metrics.

Example:

```text
HTTP 404
   ↓
Metric Filter
   ↓
404Errors = 1
```

---

## 4. CloudWatch Alarms

Alarms monitor metrics and perform actions when defined thresholds are reached.

Example:

```text
404Errors >= 5
       ↓
Alarm
       ↓
SNS
       ↓
Email
```

---

## 5. Amazon SNS

SNS provides a way to distribute notifications to subscribers.

In this lab:

```text
CloudWatch / EventBridge
        ↓
       SNS
        ↓
      Email
```

---

## 6. Amazon EventBridge

EventBridge detects events and routes them to targets.

Example:

```text
EC2 stopped
      ↓
EventBridge
      ↓
SNS
      ↓
Email
```

---

## 7. AWS Config

AWS Config evaluates AWS resource configurations against compliance rules.

Example:

```text
EC2
 ↓
required-tags
 ↓
Compliant / Noncompliant
```

---

# CloudWatch vs AWS Config

One of the most important concepts from this lab:

| CloudWatch              | AWS Config                    |
| ----------------------- | ----------------------------- |
| Performance monitoring  | Configuration monitoring      |
| Metrics                 | Resource configuration        |
| Logs                    | Compliance                    |
| Alarms                  | Configuration rules           |
| Operational health      | Audit                         |
| "How is it performing?" | "Is it configured correctly?" |

---

# Interview Questions I Can Answer After This Lab

### What is Amazon CloudWatch?

Amazon CloudWatch is an AWS monitoring and observability service used to collect metrics, logs, and events and create alarms and dashboards.

### What is the CloudWatch Agent?

The CloudWatch Agent runs on an EC2 instance or server and collects additional system metrics and logs that are not available through standard monitoring.

### What is a CloudWatch metric filter?

A metric filter searches log data for specific patterns and converts matching log events into CloudWatch metrics.

### How can logs trigger an alarm?

CloudWatch Logs can use a metric filter to create a metric. A CloudWatch alarm can then monitor that metric and trigger an action such as an SNS notification.

### What is Amazon SNS?

Amazon SNS is a messaging service that can distribute notifications to subscribers such as email endpoints.

### What is EventBridge?

Amazon EventBridge is a serverless event bus that detects events and routes them to targets such as SNS or Lambda.

### What is AWS Config?

AWS Config records and evaluates AWS resource configurations and determines whether resources comply with defined rules.

### What is the difference between CloudWatch and AWS Config?

CloudWatch focuses on operational monitoring such as metrics, logs, and alarms, while AWS Config focuses on resource configuration and compliance.

---

# What I Learned Overall

This lab gave me practical experience building an AWS monitoring and observability solution.

The most important workflow I learned was:

```text
                APPLICATION
                     |
                     v
                EC2 SERVER
                     |
                     v
             CLOUDWATCH AGENT
                     |
            +--------+--------+
            |                 |
           LOGS            METRICS
            |                 |
            v                 v
     CLOUDWATCH LOGS   CLOUDWATCH METRICS
            |                 |
            v                 v
      METRIC FILTER        ALARM
            |                 |
            +--------+--------+
                     |
                     v
                    SNS
                     |
                     v
                   EMAIL
```

I also learned how EventBridge can react to infrastructure events:

```text
EC2 STATE CHANGE
       |
       v
EVENTBRIDGE
       |
       v
SNS
       |
       v
EMAIL
```

And how AWS Config can continuously evaluate infrastructure:

```text
AWS RESOURCES
      |
      v
AWS CONFIG
      |
      v
COMPLIANCE RULES
      |
      +----> COMPLIANT
      |
      +----> NONCOMPLIANT
```

Overall, this lab strengthened my understanding of **AWS monitoring, observability, centralized logging, alerting, event-driven architecture, and infrastructure compliance**.


# Conclusion

This project demonstrates a complete AWS monitoring workflow using multiple AWS services.

The lab provided hands-on experience with:

```text
EC2
 ↓
Systems Manager
 ↓
CloudWatch Agent
 ↓
CloudWatch Logs / Metrics
 ↓
Metric Filters
 ↓
CloudWatch Alarms
 ↓
SNS
 ↓
Email Notifications

EC2 Events
 ↓
EventBridge
 ↓
SNS
 ↓
Email

AWS Resources
 ↓
AWS Config
 ↓
Compliance Monitoring
```

This project is part of my AWS hands-on learning portfolio and demonstrates practical experience with **Cloud Monitoring, Logging, Alerting, Event-Driven Architecture, and Infrastructure Compliance**.

