
# Build Amazon VPC with Public and Private Subnets from Scratch

## 📌 Overview

This hands-on lab demonstrates how to build an **Amazon Virtual Private Cloud (VPC)** from scratch without using the VPC Wizard.

All required networking components were created and configured manually, including the VPC, public and private subnets, Internet Gateway, and route tables.


## 🎯 Objectives

In this lab, I:

* Created an Amazon VPC from scratch
* Configured a VPC CIDR block
* Created a public subnet
* Created a private subnet
* Used different Availability Zones
* Created and attached an Internet Gateway
* Created custom route tables
* Associated route tables with subnets
* Configured internet routing for the public subnet
* Compared public and private subnet configurations
* Validated the VPC architecture

---

## 🏗️ Architecture

```text
                         AWS Region
                         us-east-1
                             │
                             │
                     ┌───────┴───────┐
                     │     MyVPC      │
                     │ 10.0.0.0/16    │
                     │                │
          ┌──────────┴──────┐  ┌──────┴──────────┐
          │                 │  │                 │
          │ Public Subnet   │  │ Private Subnet  │
          │ 10.0.1.0/24     │  │ 10.0.2.0/24     │
          │ us-east-1a      │  │ us-east-1b      │
          │                 │  │                 │
          │ Public Route    │  │ Private Route   │
          │ Table           │  │ Table           │
          │                 │  │                 │
          │ 0.0.0.0/0       │  │ No Internet     │
          │       ↓         │  │ Route           │
          │      IGW        │  │                 │
          └────────┬────────┘  └─────────────────┘
                   │
                   ▼
                Internet
```

---

## ☁️ AWS Services Used

* Amazon VPC
* Amazon VPC Subnets
* Amazon VPC Route Tables
* Internet Gateway
* Availability Zones
* CIDR / IPv4 Addressing

---

## ⚙️ VPC Configuration

### VPC

| Configuration | Value                             |
| ------------- | --------------------------------- |
| Name          | MyVPC                             |
| IPv4 CIDR     | 10.0.0.0/16                       |
| IPv6          | Disabled                          |
| Region        | US East (N. Virginia) – us-east-1 |
| Tenancy       | Default                           |

---

## 🌐 Subnet Configuration

Two subnets were created manually within the VPC.

| Subnet          | CIDR        | Availability Zone | Type    |
| --------------- | ----------- | ----------------- | ------- |
| MyPublicSubnet  | 10.0.1.0/24 | us-east-1a        | Public  |
| MyPrivateSubnet | 10.0.2.0/24 | us-east-1b        | Private |

The subnet CIDR blocks are non-overlapping and fall within the VPC CIDR range.

---

## 🌍 Internet Gateway

An Internet Gateway named:

```text
MyInternetGateway
```

was created and attached to `MyVPC`.

The Internet Gateway provides a path between the VPC and the internet when the appropriate route is configured.

---

## 🛣️ Route Tables

Two custom route tables were created.

### Public Route Table

```text
Name: PublicRouteTable
```

The public subnet was associated with this route table.

The following route was added:

```text
Destination: 0.0.0.0/0
Target: Internet Gateway
```

This provides a route from the public subnet to the internet.

### Private Route Table

```text
Name: PrivateRouteTable
```

The private subnet was associated with this route table.

The private route table contains the local VPC route but does not contain a direct route to the Internet Gateway.

Therefore, resources in the private subnet do not have direct internet access through the Internet Gateway.

---

## 🔍 Public vs. Private Subnet

The main difference demonstrated in this lab is routing.

### Public Subnet

```text
Resource
   ↓
Public Subnet
   ↓
Public Route Table
   ↓
0.0.0.0/0
   ↓
Internet Gateway
   ↓
Internet
```

### Private Subnet

```text
Resource
   ↓
Private Subnet
   ↓
Private Route Table
   ↓
No direct Internet Gateway route
```

A subnet is considered public when its associated route table provides a route to an Internet Gateway.

A private subnet does not have a direct route to an Internet Gateway.

---


## 🧠 Key Concepts Learned

### VPC

A VPC is a logically isolated virtual network in AWS where AWS resources can be launched.

### CIDR

The VPC was configured with:

```text
10.0.0.0/16
```

This provides the IP address range used by the VPC.

### Subnet

A subnet is a segment of the VPC IP address range.

In this lab:

```text
VPC
10.0.0.0/16
│
├── Public Subnet
│   └── 10.0.1.0/24
│
└── Private Subnet
    └── 10.0.2.0/24
```

### Route Table

Route tables contain rules that determine where network traffic is directed.

Each subnet is associated with one route table.

### Internet Gateway

An Internet Gateway allows communication between a VPC and the internet when the subnet's route table contains an appropriate route.

---

## 🔑 Key Takeaways

This lab helped me understand that creating a VPC is more than simply creating a network.

The main components work together:

```text
VPC
 │
 ├── Subnets
 │
 ├── Route Tables
 │
 └── Internet Gateway
```

The routing configuration determines whether a subnet is public or private.

The most important concept from this lab was:

```text
Public Subnet
     ↓
Route Table
     ↓
Internet Gateway
     ↓
Internet
```

while the private subnet has no direct route to the Internet Gateway.

---

## 🛠️ Skills Demonstrated

* AWS VPC configuration
* IPv4 addressing
* CIDR
* Subnetting
* Public and private subnet design
* Availability Zones
* Route table configuration
* Internet Gateway
* AWS networking fundamentals
* Cloud architecture



This lab was completed as part of my hands-on AWS cloud training.

**Status:** ✅ Completed
