# AWS CloudWatch Dashboard for EC2 Instance

## 📌 Overview

This hands-on AWS lab demonstrates how to launch an **Amazon EC2 instance** and create a centralized **Amazon CloudWatch dashboard** to monitor its CPU utilization.

I created multiple CloudWatch widgets using the same EC2 `CPUUtilization` metric, including:

* Line chart
* Stacked area chart
* Number widget

This lab provided practical experience with EC2 monitoring, CloudWatch metrics, dashboards, and infrastructure observability.

---

## 🎯 Objectives

By completing this lab, I learned how to:

* Launch an Amazon EC2 instance
* Configure EC2 networking and security
* Access EC2 instance monitoring metrics
* Locate EC2 `CPUUtilization` metrics in CloudWatch
* Create a CloudWatch dashboard
* Add multiple widget types
* Monitor the same metric using different visualizations
* Use CloudWatch as a centralized monitoring solution

---

## ☁️ AWS Services Used

| AWS Service       | Purpose                            |
| ----------------- | ---------------------------------- |
| Amazon EC2        | Compute resource being monitored   |
| Amazon CloudWatch | Metrics, monitoring, and dashboard |
| AWS IAM           | Authentication and permissions     |
| Amazon VPC        | Networking for the EC2 instance    |

---

## 🌎 AWS Region

**Region:** US East (N. Virginia)

```text
us-east-1
```

---

# 🏗️ Architecture

```text
                ┌──────────────────────┐
                │      Amazon EC2      │
                │                      │
                │   learner-labs       │
                │                      │
                │  CPUUtilization      │
                └──────────┬───────────┘
                           │
                           │ EC2 Metrics
                           ▼
                ┌──────────────────────┐
                │   Amazon CloudWatch  │
                │                      │
                │   EC2 Metrics        │
                └──────────┬───────────┘
                           │
                           ▼
                ┌──────────────────────┐
                │ CloudWatch Dashboard │
                │                      │
                │ ┌──────────────────┐ │
                │ │ Line             │ │
                │ ├──────────────────┤ │
                │ │ Stacked Area     │ │
                │ ├──────────────────┤ │
                │ │ Number           │ │
                │ └──────────────────┘ │
                └──────────────────────┘
```

---

# 1. Launch EC2 Instance

I launched an Amazon EC2 instance using **Amazon Linux 2023**.

### Configuration

```text
Instance Name: learner-labs
AMI: Amazon Linux 2023
Architecture: 64-bit (x86)
Instance Type: t2.micro
Region: us-east-1
```

A key pair was created for secure SSH access.

### Security Group

```text
Security Group: MyEC2Server_SG
Inbound Rule: SSH
Port: 22
```

> For production environments, SSH access should be restricted to trusted IP addresses rather than allowing access from anywhere.

---

# 2. Verify EC2 Instance

After launching the instance, I verified that the instance reached the:

```text
Instance state: Running
```

I also recorded:

* Instance ID
* Public IPv4 address
* Instance type
* Availability Zone

The EC2 instance generates standard monitoring metrics that can be viewed through Amazon CloudWatch.

---

# 3. Monitor EC2 with CloudWatch

I opened Amazon CloudWatch and navigated to the available EC2 metrics.

The primary metric used in this lab was:

```text
CPUUtilization
```

### Metric Details

```text
Namespace: AWS/EC2
Metric: CPUUtilization
Dimension: InstanceId
```

CloudWatch collects standard EC2 metrics automatically. There can be a short delay before newly launched instances begin showing metric data.

---

# 4. Create CloudWatch Dashboard

I created a CloudWatch dashboard named:

```text
MyEC2
```

The dashboard provides a centralized view of the EC2 instance's CPU utilization.

---

# 5. Add Line Chart Widget

The first widget was a **Line** chart.

### Configuration

```text
Metric: CPUUtilization
Namespace: AWS/EC2
Dimension: InstanceId
Instance: learner-labs
```

The line chart provides a time-series view of CPU utilization.

---

# 6. Add Stacked Area Widget

I added a second widget using the same:

```text
CPUUtilization
```

metric.

The widget type was:

```text
Stacked Area
```

This demonstrates how the same CloudWatch metric can be visualized using different chart types.

---

# 7. Add Number Widget

I added a third widget using:

```text
CPUUtilization
```

The widget type was:

```text
Number
```

The Number widget provides a simple view of the current/latest metric value.

---

# 📊 Final Dashboard

The completed dashboard contains multiple visualizations of the same EC2 CPU metric:

```text
┌──────────────────────────────────────────────┐
│              MyEC2 Dashboard                 │
├──────────────────────────────────────────────┤
│                                              │
│             CPUUtilization                   │
│              Line Chart                      │
│                                              │
├──────────────────────────────────────────────┤
│                                              │
│             CPUUtilization                   │
│            Stacked Area                      │
│                                              │
├──────────────────────────────────────────────┤
│                                              │
│              CPUUtilization                  │
│                  12.4%                       │
│                Number                        │
│                                              │
└──────────────────────────────────────────────┘
```

> The actual CPU utilization value changes depending on EC2 workload and activity.

---

# 🧪 Validation

The lab was successfully completed when:

* [x] EC2 instance was launched
* [x] Amazon Linux 2023 was selected
* [x] Security group was configured
* [x] EC2 instance reached the Running state
* [x] EC2 `CPUUtilization` metric was located
* [x] CloudWatch dashboard was created
* [x] Line widget was added
* [x] Stacked Area widget was added
* [x] Number widget was added
* [x] Dashboard was saved
* [x] CPU utilization data was displayed

---

# 🧠 Key Concepts Learned

### Amazon CloudWatch

CloudWatch provides monitoring and observability capabilities for AWS resources and applications.

### CloudWatch Metrics

Metrics are numerical measurements that can be used to monitor resource performance.

For EC2, an important standard metric is:

```text
CPUUtilization
```

### CloudWatch Dashboards

Dashboards provide a centralized visual representation of monitoring data.

### EC2 Monitoring

CloudWatch allows administrators and engineers to monitor EC2 performance and identify unusual resource utilization.

### Data Visualization

The same metric can be represented using different widget types depending on the monitoring requirement.

---

# 🔐 Security Considerations

The lab security group allowed SSH access for demonstration purposes.

For production environments:

* Restrict SSH access to trusted IP addresses
* Avoid `0.0.0.0/0` for SSH whenever possible
* Use AWS Systems Manager Session Manager when appropriate
* Follow the principle of least privilege
* Remove unused security groups and resources

---

# 📸 Screenshots

The following screenshots document the implementation.

### 1. EC2 Instance

Show:

* Instance name
* Instance ID
* Instance state: **Running**
* Instance type
* Availability Zone

### 2. EC2 Monitoring / Metrics

Show the EC2 instance and its monitoring information.

### 3. CloudWatch CPUUtilization Metric

Show:

```text
AWS/EC2
→ Per-Instance Metrics
→ CPUUtilization
```

with the correct Instance ID.

### 4. CloudWatch Dashboard

Show the dashboard name:

```text
MyEC2
```

and the first Line widget.

### 5. Stacked Area Widget

Show the `CPUUtilization` metric displayed as a Stacked Area chart.

### 6. Number Widget

Show the current/latest `CPUUtilization` value.

### 7. Final Dashboard ⭐

This is the most important screenshot.

Show all three widgets together:

```text
Line
Stacked Area
Number
```

The dashboard should clearly show that all widgets are monitoring the same EC2 `CPUUtilization` metric.



# 🧹 Cleanup

After completing the lab, I deleted the resources that were created specifically for this exercise to avoid unnecessary AWS charges.

Recommended cleanup:

* Terminate the EC2 instance
* Delete the CloudWatch dashboard if it is no longer needed
* Delete the lab-specific security group
* Delete the lab-specific key pair if no longer required

---

# 💡 What This Lab Demonstrates

This project demonstrates practical experience with:

```text
EC2
  │
  ▼
CloudWatch Metrics
  │
  ▼
CPUUtilization
  │
  ▼
CloudWatch Dashboard
  │
  ├── Line Chart
  ├── Stacked Area
  └── Number Widget
```

It shows how AWS infrastructure can be monitored through **CloudWatch metrics and dashboards** and provides hands-on experience with basic AWS observability.

---

# 📚 AWS Services

* Amazon EC2
* Amazon CloudWatch
* AWS IAM
* Amazon VPC
