# Introduction to Amazon Elastic Compute Cloud (EC2)

## Overview

This hands-on lab demonstrates how to launch and configure an **Amazon EC2 instance** and deploy a basic Apache web server.

The lab covers the use of an **Amazon Machine Image (AMI)**, EC2 instance configuration, key pair authentication, Security Groups, Linux commands, Apache installation, and web page deployment.

As part of the lab, a custom HTML page was created and published through the Apache web server.



## Objectives

* Launch an Amazon EC2 instance
* Select and use an Amazon Machine Image (AMI)
* Configure an EC2 instance
* Create and use a key pair
* Configure a Security Group
* Connect to an EC2 instance using EC2 Instance Connect
* Install and configure Apache HTTP Server
* Start and enable the Apache service
* Create a custom HTML page
* Publish the page through a web server
* Access the web server using the EC2 public IP address

---

## AWS Region

```text
Region: US East (N. Virginia)
Region Code: us-east-1
```

---

## AWS Services and Technologies

* Amazon EC2
* Amazon Machine Images (AMI)
* Amazon EBS
* Amazon VPC
* Security Groups
* Key Pairs
* EC2 Instance Connect
* Apache HTTP Server
* Linux
* HTML

---

## Architecture

```text
                    Internet
                       │
                       │ HTTP :80
                       │
                Public IPv4 Address
                       │
                       ▼
              ┌──────────────────┐
              │   EC2 Instance   │
              │   MyEC2Server    │
              │                  │
              │  Amazon Linux 2  │
              │                  │
              │ Apache (httpd)   │
              │       :80        │
              └────────┬─────────┘
                       │
                       ▼
               /var/www/html/
                       │
                       ▼
                   test.html
```

---

## EC2 Configuration

| Configuration    | Value                  |
| ---------------- | ---------------------- |
| Instance Name    | `MyEC2Server`          |
| Operating System | Amazon Linux 2         |
| Region           | `us-east-1`            |
| Instance Type    | `t2.micro`             |
| Public IP        | Assigned automatically |
| Storage          | Default EBS            |
| Web Server       | Apache HTTP Server     |

---

## Security Group

Security Group:

```text
MyEC2Server_SG
```

### Inbound Rules

| Type | Protocol | Port | Source      |
| ---- | -------- | ---: | ----------- |
| SSH  | TCP      |   22 | `0.0.0.0/0` |
| HTTP | TCP      |   80 | `0.0.0.0/0` |

The SSH rule allows remote administrative access, while the HTTP rule allows users to access the web server.

> **Security Note:** The lab uses `0.0.0.0/0` for demonstration purposes. In a production environment, SSH access should be restricted to trusted IP addresses or replaced with a more secure access method such as AWS Systems Manager Session Manager.

---

## Key Pair

A new RSA key pair was created:

```text
MyEC2Key
```

The private key was downloaded and stored locally for secure authentication.

> **Important:** The private key file (`.pem`) must never be uploaded to GitHub.

---

## Connecting to the EC2 Instance

EC2 Instance Connect was used to establish a connection to the instance.

After connecting, the Linux environment was accessed through the browser-based terminal.

Example:

```bash
sudo su
```

---

## Installing Apache

The system packages were updated:

```bash
yum -y update
```

Apache HTTP Server was installed:

```bash
yum install httpd -y
```

The Apache service was started:

```bash
systemctl start httpd
```

Apache was configured to start automatically after a reboot:

```bash
systemctl enable httpd
```

The service status was verified:

```bash
systemctl status httpd
```

Expected status:

```text
Active: active (running)
```

---

## Testing the Web Server

The Apache web server was tested by opening the EC2 public IPv4 address in a web browser:

```text
http://<PUBLIC-IP>
```

The Apache test page confirmed that the web server was successfully installed and running.

---

## Creating a Custom HTML Page

The Apache web root directory was accessed:

```bash
cd /var/www/html/
```

A custom HTML file was created:

```bash
nano test.html
```

Example content:

```html
<HTML>
Hi Bamlabs, I am a public page
</HTML>
```

The Apache service was restarted:

```bash
systemctl restart httpd
```

The custom page was then accessed through:

```text
http://<PUBLIC-IP>/test.html
```

---

## Result

The custom HTML page was successfully published through the Apache web server running on the EC2 instance.

```text
Internet
    │
    ▼
Public IPv4
    │
    ▼
EC2 Instance
    │
    ▼
Apache HTTP Server
    │
    ▼
/var/www/html/test.html
```
---

## Key Concepts Learned

### Amazon EC2

Amazon EC2 provides resizable compute capacity in the AWS Cloud. EC2 instances are virtual servers that can run applications and services.

### AMI

An **Amazon Machine Image (AMI)** provides the software configuration required to launch an EC2 instance, including the operating system.

### Key Pair

A key pair is used for secure authentication when connecting to an EC2 instance.

### Security Group

A Security Group acts as a **stateful virtual firewall** controlling inbound and outbound traffic for an EC2 instance.

### EBS

Amazon Elastic Block Store (EBS) provides persistent block storage for EC2 instances.

### Apache

Apache HTTP Server is a web server used to serve web content over HTTP.

---

## Skills Demonstrated

**AWS:**

* Amazon EC2
* AMI
* EBS
* VPC
* Security Groups
* Key Pairs
* EC2 Instance Connect

**Linux:**

* Package management
* `sudo`
* `yum`
* `systemctl`
* Linux directories
* Service management

**Web Technologies:**

* Apache HTTP Server
* HTTP
* HTML
* Web server deployment

---

## Key Takeaways

* Learned how to launch an EC2 instance using an AMI.
* Configured an EC2 Security Group.
* Connected to an EC2 instance using EC2 Instance Connect.
* Installed and configured Apache HTTP Server.
* Managed Linux services using `systemctl`.
* Created and deployed a custom HTML page.
* Published a web application using an EC2 public IP address.
* Practiced basic AWS compute and networking concepts.


**Lab:** Introduction to Amazon Elastic Compute Cloud (EC2)

**Status:** Completed
