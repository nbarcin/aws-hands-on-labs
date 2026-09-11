# Allocating an Elastic IP and Associating It with an EC2 Instance

## Overview

This hands-on lab demonstrates how to allocate an **Elastic IP address** in AWS and associate it with an Amazon EC2 instance.

The lab builds on basic EC2 and web server concepts by launching an EC2 instance, configuring an Apache web server, publishing a custom HTML page, and then replacing the instance's initial public IPv4 address with an Elastic IP address.


## Objectives

* Launch an Amazon EC2 instance
* Configure an EC2 Security Group
* Connect to an EC2 instance
* Install and configure Apache HTTP Server
* Create and publish a custom HTML page
* Test a web server using an EC2 public IPv4 address
* Allocate an Elastic IP address
* Associate an Elastic IP with an EC2 instance
* Access the web server using the Elastic IP address

---

## AWS Region

```text
Region: US East (N. Virginia)
Region Code: us-east-1
```

---

## AWS Services and Technologies

* Amazon EC2
* Elastic IP
* Amazon Machine Image (AMI)
* Amazon EBS
* Amazon VPC
* Security Groups
* Key Pairs
* EC2 Instance Connect / SSH
* Apache HTTP Server
* Linux
* HTML
* IPv4

---

## Architecture

### Before Elastic IP

```text
                    Internet
                       │
                       │ HTTP :80
                       ▼
                EC2 Public IPv4
                       │
                       ▼
              ┌──────────────────┐
              │   MyEC2Server    │
              │                  │
              │  Amazon Linux 2  │
              │                  │
              │ Apache HTTP :80  │
              └────────┬─────────┘
                       │
                       ▼
                  test.html
```

### After Elastic IP

```text
                    Internet
                       │
                       │ HTTP :80
                       ▼
                 Elastic IP
                       │
                       ▼
              ┌──────────────────┐
              │   MyEC2Server    │
              │                  │
              │  Amazon Linux 2  │
              │                  │
              │ Apache HTTP :80  │
              └────────┬─────────┘
                       │
                       ▼
                  test.html
```

---

## EC2 Configuration

| Configuration    | Value              |
| ---------------- | ------------------ |
| Instance Name    | `MyEC2Server`      |
| Operating System | Amazon Linux 2     |
| Region           | `us-east-1`        |
| Instance Type    | `t2.micro`         |
| Web Server       | Apache HTTP Server |
| Security Group   | `whizlabs_SG`      |
| Storage          | Default EBS        |

---

## Security Group

The EC2 instance was configured with a Security Group allowing web and administrative traffic.

### Inbound Rules

| Type | Protocol | Port | Source      |
| ---- | -------- | ---: | ----------- |
| SSH  | TCP      |   22 | `0.0.0.0/0` |
| HTTP | TCP      |   80 | `0.0.0.0/0` |

> **Security Note:** `0.0.0.0/0` was used for this hands-on lab. In a production environment, SSH access should be restricted to trusted IP addresses or replaced with a more secure access method.

---

## Apache Web Server Configuration

The EC2 instance was accessed through SSH / EC2 Instance Connect.

The system packages were updated:

```bash
yum -y update
```

Apache was installed:

```bash
yum install httpd -y
```

The Apache service was started:

```bash
systemctl start httpd
```

Apache was enabled to start automatically:

```bash
systemctl enable httpd
```

The service status was verified:

```bash
systemctl status httpd
```

Expected result:

```text
Active: active (running)
```

---

## Creating the Web Page

The Apache web root directory was accessed:

```bash
cd /var/www/html/
```

A custom HTML file was created:

```bash
nano test.html
```

Example:

```html
<HTML>
Hi Whizlabs, I am a public page
</HTML>
```

The Apache service was restarted:

```bash
systemctl restart httpd
```

The page was initially tested using the EC2 public IPv4 address:

```text
http://<PUBLIC-IP>/test.html
```

---

## Elastic IP

An Elastic IP address was allocated from the EC2 console.

The Elastic IP was then associated with:

```text
MyEC2Server
```

After the association, the EC2 instance used the Elastic IP as its public IPv4 address.

### Traffic Flow

```text
Client
  │
  ▼
Elastic IP
  │
  ▼
EC2 Instance
  │
  ▼
Apache Web Server
  │
  ▼
test.html
```

---

## Testing the Elastic IP

After associating the Elastic IP with the EC2 instance, the web page was tested again.

```text
http://<ELASTIC-IP>/test.html
```

The custom HTML page was successfully displayed.

This confirmed that the EC2 instance was reachable through its Elastic IP and that the Apache web server was serving the requested content.

---

## Elastic IP vs Dynamic Public IP

| Feature                   | Public IPv4            | Elastic IP                                |
| ------------------------- | ---------------------- | ----------------------------------------- |
| Address type              | Dynamic                | Static                                    |
| Can change                | Yes                    | No, while allocated                       |
| Associated with EC2       | Yes                    | Yes                                       |
| Useful for fixed endpoint | Limited                | Yes                                       |
| Can be remapped           | Limited                | Yes                                       |
| Cost consideration        | Depends on AWS pricing | Charges can apply, especially when unused |

An Elastic IP is useful when an application needs a **stable public IPv4 address**.

---

## When Would You Use an Elastic IP?

Typical use cases include:

* Hosting a service that requires a fixed public IP
* Maintaining a stable endpoint for an application
* DNS configurations that point to a fixed IP
* Reassociating a public IP with another EC2 instance during recovery
* Applications where changing the public IP would cause disruption

For many modern AWS architectures, services such as **Application Load Balancer** or **Route 53** may be preferable instead of assigning an Elastic IP directly to an application server.



## Key Takeaways

* Learned how to launch and configure an EC2 instance.
* Installed and configured Apache HTTP Server.
* Created and published a custom HTML page.
* Tested a web server using a public IPv4 address.
* Allocated an Elastic IP address.
* Associated an Elastic IP with an EC2 instance.
* Verified the Elastic IP through the EC2 console.
* Tested the web application using the Elastic IP.
* Learned when a static public IP can be useful.

---

## Skills Demonstrated

**AWS:**

* Amazon EC2
* Elastic IP
* AMI
* EBS
* VPC
* Security Groups
* Key Pairs

**Networking:**

* Public IPv4
* Static IP
* Dynamic IP
* HTTP
* TCP
* Port 22
* Port 80

**Linux:**

* `yum`
* `systemctl`
* Apache configuration
* Linux web server deployment

**Web:**

* Apache HTTP Server
* HTML
* HTTP web hosting
* 
**Lab:** Allocating Elastic IP and Associating It to an EC2 Instance

**Status:** Completed
