# Understanding and Configuring Layered Security in an AWS VPC

## Overview

This hands-on lab demonstrates how to configure **multi-layered security in an Amazon VPC** using **Security Groups and Network Access Control Lists (NACLs)**.

The lab also demonstrates how to create public and private subnets, configure routing, launch EC2 instances, and test network connectivity between resources.


## Objectives

* Create a custom Amazon VPC
* Configure a VPC CIDR block
* Create public and private subnets
* Create and attach an Internet Gateway
* Configure public and private route tables
* Create and configure a Security Group
* Create and configure a Network ACL
* Associate NACLs with subnets
* Launch EC2 instances in public and private subnets
* Configure layered network security
* Test connectivity between EC2 instances using ICMP

---

## Architecture

```text
                         Internet
                            │
                            │
                    Internet Gateway
                            │
                    ┌───────┴────────┐
                    │     whizlabs    │
                    │      VPC        │
                    │   10.0.0.0/16   │
                    │                 │
                    │   Network ACL   │
                    │       │         │
            ┌───────┴───────┴───────┐
            │                       │
      Public Subnet          Private Subnet
       10.0.1.0/24            10.0.2.0/24
            │                       │
       Public EC2              Private EC2
            │                       │
       Public IP                 Private IP
            │                       │
            └─────── ICMP ─────────┘
```

---

## AWS Region

```text
Region: US East (N. Virginia)
Region Code: us-east-1
```

---

## AWS Services Used

* Amazon VPC
* Amazon EC2
* Internet Gateway
* Route Tables
* Security Groups
* Network ACLs
* Availability Zones
* IPv4 / CIDR

---

## VPC Configuration

| Resource  | Configuration  |
| --------- | -------------- |
| VPC Name  | `whizlabs_VPC` |
| IPv4 CIDR | `10.0.0.0/16`  |
| IPv6      | Disabled       |
| Tenancy   | Default        |
| Region    | `us-east-1`    |

---

## Subnet Configuration

| Subnet           | CIDR          | Purpose           |
| ---------------- | ------------- | ----------------- |
| `public_subnet`  | `10.0.1.0/24` | Public resources  |
| `private_subnet` | `10.0.2.0/24` | Private resources |

The public subnet is associated with a route table that contains a default route to the Internet Gateway.

The private subnet does not have a direct route to the Internet Gateway.

---

## Internet Gateway

Created:

```text
whizlabs_IGW
```

The Internet Gateway was attached to:

```text
whizlabs_VPC
```

The Internet Gateway provides a path between the VPC and the public internet for resources configured with appropriate routing and public IP addressing.

---

## Route Tables

### Public Route Table

```text
Route Table: public_route

Destination        Target
--------------------------------
10.0.0.0/16        local
0.0.0.0/0          whizlabs_IGW
```

The public subnet is associated with this route table.

### Private Route Table

```text
Route Table: private_route

Destination        Target
--------------------------------
10.0.0.0/16        local
```

The private subnet is associated with this route table.

There is no direct Internet Gateway route from the private subnet.

---

## Security Group

Security Group:

```text
whizlabs_securitygroup
```

The Security Group provides **instance-level stateful traffic filtering**.

### Inbound Rules

```text
SSH
Port: 22
Source: 0.0.0.0/0

All ICMP - IPv4
Source: 0.0.0.0/0
```

The rules were configured for the purposes of this hands-on lab.

> Security best practice: In a production environment, SSH access should be restricted to trusted IP addresses or secured through alternatives such as AWS Systems Manager Session Manager.

---

## Network ACL

Network ACL:

```text
whizlabs_NACL
```

The NACL was associated with both:

```text
public_subnet
private_subnet
```

### Inbound Rules

```text
Rule 100    SSH                  ALLOW
Rule 200    All ICMP - IPv4      ALLOW
```

### Outbound Rules

```text
Rule 100    SSH                  ALLOW
Rule 200    All ICMP - IPv4      ALLOW
Rule 300    TCP 1024-65535       ALLOW
```

Network ACLs operate at the **subnet level** and are **stateless**, which means inbound and outbound traffic must be explicitly allowed.

---

## EC2 Instances

Two EC2 instances were launched:

| Instance           | Subnet         | IP Type             |
| ------------------ | -------------- | ------------------- |
| `public_instance`  | Public subnet  | Public + Private IP |
| `private_instance` | Private subnet | Private IP          |

Both instances were deployed inside:

```text
whizlabs_VPC
```

and associated with:

```text
whizlabs_securitygroup
```

---

## Connectivity Test

After connecting to the public EC2 instance using SSH, the private IP address of the private EC2 instance was tested using `ping`.

Example:

```bash
ping <private-ec2-private-ip>
```

Expected result:

```text
64 bytes from 10.0.2.x: icmp_seq=1 ttl=64 time=...
64 bytes from 10.0.2.x: icmp_seq=2 ttl=64 time=...
```

### Result

The private EC2 instance successfully responded to ICMP traffic.

This demonstrated that the configured **Security Group, Network ACL, subnet association, and VPC networking rules** allowed the traffic between the two instances.

---

## Security Layers

This lab demonstrates multiple layers of network security:

```text
                VPC
                 │
          ┌──────┴──────┐
          │             │
       Routing        NACL
          │             │
          │        Subnet Level
          │
        Security Group
        Instance Level
          │
         EC2
```

### Security Group vs NACL

| Feature         | Security Group | Network ACL      |
| --------------- | -------------- | ---------------- |
| Level           | Instance       | Subnet           |
| Stateful        | Yes            | No               |
| Inbound rules   | Yes            | Yes              |
| Outbound rules  | Yes            | Yes              |
| Rule evaluation | Allow rules    | Allow/Deny rules |
| Associated with | ENI/Instance   | Subnet           |

---

## Key Concepts Learned

### VPC

A VPC provides an isolated virtual network in AWS where cloud resources can be deployed and controlled.

### Public Subnet

A subnet is considered public when its route table provides a route to an Internet Gateway.

### Private Subnet

A private subnet does not have a direct route to an Internet Gateway.

### Security Group

A Security Group acts as a **stateful virtual firewall at the instance level**.

### Network ACL

A Network ACL acts as a **stateless firewall at the subnet level**.

### Layered Security

Using multiple security controls provides defense in depth.

In this lab:

```text
Internet
   ↓
Internet Gateway
   ↓
Route Table
   ↓
Network ACL
   ↓
Security Group
   ↓
EC2 Instance
```

---

## Key Takeaways

* Learned how to build a custom VPC from scratch.
* Created public and private subnets.
* Configured Internet Gateway and route tables.
* Learned the difference between Security Groups and Network ACLs.
* Configured stateful instance-level security using Security Groups.
* Configured stateless subnet-level security using NACLs.
* Launched EC2 instances in different subnets.
* Tested private network connectivity using ICMP.
* Practiced the concept of **defense in depth** in AWS networking.

---

## Skills Demonstrated

**AWS:** VPC, EC2, Internet Gateway, Route Tables, Security Groups, NACLs

**Networking:** IPv4, CIDR, subnets, routing, public/private networks, ICMP

**Security:** Stateful firewall, stateless firewall, layered security, defense in depth

**Cloud:** AWS networking and infrastructure configuration

---

Lab Focus:

> Multi-Layered VPC Security and EC2 Network Connectivity

**Status:** Completed
