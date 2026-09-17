# Monitor an EC2 Instance with Amazon CloudWatch, SNS & Systems Manager

## Overview

This hands-on AWS lab demonstrates how to monitor an Amazon EC2 instance using Amazon CloudWatch, create an automated alert with Amazon SNS, and securely connect to the instance using AWS Systems Manager Session Manager.

The lab simulates a high-CPU event on an EC2 instance and demonstrates how CloudWatch detects the event and sends an SNS email notification.

## Objectives

By completing this lab, I:

* Created an Amazon SNS topic and email subscription
* Created a CloudWatch CPU utilization alarm
* Connected to an EC2 instance using Systems Manager Session Manager
* Generated CPU load on the EC2 instance
* Triggered a CloudWatch alarm
* Verified an SNS email notification
* Created a CloudWatch dashboard
* Monitored EC2 CPU utilization

## AWS Services Used

* Amazon EC2
* Amazon CloudWatch
* Amazon SNS
* AWS Systems Manager Session Manager
* AWS IAM

## Architecture

```text
                  Amazon EC2
                Stress Test VM
                     │
                     │ CPUUtilization
                     ▼
              Amazon CloudWatch
                     │
               CPU > 60%
                     │
                     ▼
              CloudWatch Alarm
                     │
                     ▼
                 Amazon SNS
                     │
                     ▼
                Email Alert


       Systems Manager Session Manager
                     │
                     ▼
                  Amazon EC2
                     │
                CPU Stress Test
```

## Lab Configuration

| Resource          | Configuration                   |
| ----------------- | ------------------------------- |
| SNS Topic         | `MyCwAlarm`                     |
| SNS Protocol      | Email                           |
| CloudWatch Metric | `CPUUtilization`                |
| Statistic         | Average                         |
| Period            | 1 minute                        |
| Threshold         | Greater than 60%                |
| Alarm             | `LabCPUUtilizationAlarm`        |
| Dashboard         | `LabEC2Dashboard`               |
| Connection Method | Systems Manager Session Manager |

## Implementation

### 1. Create an SNS Topic

Created an SNS Standard topic:

```text
MyCwAlarm
```

Created an email subscription and confirmed the subscription from the AWS email notification.

### 2. Configure CloudWatch

Navigated to:

```text
CloudWatch
→ Metrics
→ Classic metrics
→ EC2
→ Per-Instance Metrics
```

Selected:

```text
CPUUtilization
```

for the target EC2 instance.

### 3. Create the CloudWatch Alarm

Configured the alarm with:

```text
Statistic: Average
Period: 1 minute
Threshold: CPUUtilization > 60%
Alarm Name: LabCPUUtilizationAlarm
SNS Topic: MyCwAlarm
```

The alarm monitors the EC2 CPU utilization and changes to the `ALARM` state when the configured threshold is exceeded.

### 4. Connect to EC2 with Session Manager

Instead of using SSH, I connected to the EC2 instance through:

```text
EC2
→ Instances
→ Select Instance
→ Connect
→ Session Manager
→ Connect
```

This provided browser-based shell access without requiring an SSH key.

### 5. Generate CPU Load

Inside the EC2 instance, I generated CPU load for testing.

Example:

```bash
stress --cpu 10 --timeout 400
```

I monitored the CPU utilization using:

```bash
top
```

### 6. Verify the CloudWatch Alarm

As CPU utilization increased above the configured threshold, the CloudWatch alarm transitioned to:

```text
ALARM
```

CloudWatch then triggered the SNS notification.

### 7. Verify SNS Notification

The SNS subscription sent an email notification when the CloudWatch alarm entered the `ALARM` state.

This demonstrated the complete monitoring and alerting workflow:

```text
High CPU
   ↓
CloudWatch Metric
   ↓
CloudWatch Alarm
   ↓
SNS
   ↓
Email Notification
```

### 8. Create a CloudWatch Dashboard

Created:

```text
LabEC2Dashboard
```

and added the EC2 `CPUUtilization` metric as a line graph.

The dashboard provides a centralized view of the EC2 CPU performance.

## Validation

The following results were verified:

* [x] SNS topic created
* [x] Email subscription confirmed
* [x] EC2 CPUUtilization metric available
* [x] CloudWatch alarm created
* [x] EC2 CPU utilization increased
* [x] CloudWatch alarm entered `ALARM`
* [x] SNS notification received
* [x] CloudWatch dashboard created
* [x] EC2 connected through Session Manager

## Security & Monitoring Concepts

This lab demonstrates several important AWS cloud concepts:

### Monitoring

CloudWatch provides visibility into EC2 resource performance through metrics such as CPU utilization.

### Alerting

CloudWatch alarms can automatically respond when a metric crosses a defined threshold.

### Notification

Amazon SNS provides a notification mechanism for sending alerts to subscribers.

### Secure Access

Systems Manager Session Manager provides browser-based access to managed EC2 instances without requiring inbound SSH access.

### Incident Detection

A sudden CPU spike can be an indicator of a performance problem, resource-intensive process, or potentially suspicious activity. Monitoring and alerting help administrators investigate these events quickly.

## Skills Demonstrated

* Amazon EC2
* Amazon CloudWatch
* CloudWatch Metrics
* CloudWatch Alarms
* Amazon SNS
* Systems Manager Session Manager
* IAM
* Linux
* Monitoring & Observability
* Incident Detection
* AWS Console
* Troubleshooting


## Key Takeaway

This lab demonstrates an end-to-end AWS monitoring and alerting workflow:

```text
EC2
 ↓
CloudWatch Metrics
 ↓
CloudWatch Alarm
 ↓
SNS
 ↓
Email Notification
```

It also demonstrates secure operational access using Systems Manager Session Manager.

## AWS Documentation

* Amazon CloudWatch CPU alarms
* Amazon SNS
* AWS Systems Manager Session Manager
* Amazon EC2 monitoring
