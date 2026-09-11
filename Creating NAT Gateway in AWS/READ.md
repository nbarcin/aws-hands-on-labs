# AWS NAT Gateway – Private Subnet Internet Access

## 📌 Overview

This hands-on lab demonstrates how to configure a **NAT Gateway** to provide outbound internet access to an EC2 instance running in a private subnet.

The lab was completed as part of the **AWS re/Start Cloud Computing Program**.

The main objective was to understand how public and private subnets communicate with the internet and how a NAT Gateway enables private resources to initiate outbound connections without exposing them directly to the internet.

---

## 🎯 Objectives

In this lab, I:

* Created an Amazon VPC
* Created public and private subnets
* Created and attached an Internet Gateway
* Configured a public route table
* Launched an EC2 instance in a public subnet
* Launched an EC2 instance in a private subnet
* Connected to the private EC2 instance through the public EC2 instance
* Tested internet connectivity from the private subnet
* Created a NAT Gateway
* Allocated an Elastic IP address
* Updated the private subnet route table
* Verified internet access from the private EC2 instance

---

## 🏗️ Architecture

```text
                         Internet
                             │
                             │
                    Internet Gateway
                             │
                    ┌────────┴────────┐
                    │     MyVPC       │
                    │  10.0.0.0/16    │
                    │                 │
        Public Subnet              Private Subnet
        10.0.0.0/24               10.0.1.0/24
             │                          │
             │                          │
      Public EC2                  Private EC2
             │                          │
             │                          │
             └─────── SSH ─────────────┘
                                        │
                                        │
                                  NAT Gateway
                                        │
                                        │
                                Internet Gateway
                                        │
                                        ▼
                                    Internet
```

### Traffic Flow

For the public EC2 instance:

```text
Public EC2
    ↓
Public Route Table
    ↓
Internet Gateway
    ↓
Internet
```

For the private EC2 instance:

```text
Private EC2
    ↓
Private Route Table
    ↓
NAT Gateway
    ↓
Internet Gateway
    ↓
Internet
```

---

## ☁️ AWS Services Used

* **Amazon VPC**
* **Amazon EC2**
* **NAT Gateway**
* **Internet Gateway**
* **Amazon VPC Route Tables**
* **Amazon VPC Subnets**
* **Security Groups**
* **Elastic IP**
* **Amazon Linux**

---

## ⚙️ Configuration

### VPC

| Resource  | Configuration                     |
| --------- | --------------------------------- |
| VPC Name  | MyVPC                             |
| IPv4 CIDR | 10.0.0.0/16                       |
| Region    | US East (N. Virginia) – us-east-1 |

### Subnets

| Subnet          | CIDR        | Type    |
| --------------- | ----------- | ------- |
| MyPublicSubnet  | 10.0.0.0/24 | Public  |
| MyPrivateSubnet | 10.0.1.0/24 | Private |

### Internet Gateway

```text
MyIGW
```

The Internet Gateway was attached to `MyVPC`.

### Public Route Table

```text
Destination: 0.0.0.0/0
Target: MyIGW
```

The public subnet was associated with this route table.

### NAT Gateway

```text
Name: MyNATGateway
Subnet: MyPublicSubnet
Elastic IP: Allocated
Status: Available
```

The NAT Gateway was created inside the public subnet.

### Private Route Table

The main route table was configured with:

```text
Destination: 0.0.0.0/0
Target: MyNATGateway
```

This allows instances in the private subnet to initiate outbound internet connections.

---

## 🖥️ EC2 Configuration

Two Amazon Linux EC2 instances were launched:

### Public EC2

```text
Name: MyPublicEC2Server
Subnet: MyPublicSubnet
Public IP: Enabled
```

### Private EC2

```text
Name: MyPrivateEC2Server
Subnet: MyPrivateSubnet
Public IP: Disabled
```

The private EC2 instance does not have a public IP address.

---

## 🔐 SSH Connection

I connected to the public EC2 instance first and then connected to the private EC2 instance using its private IP address.

Example:

```bash
ssh -i "MyKey.pem" ec2-user@10.0.1.45
```

The private key was stored securely and was not uploaded to GitHub.

---

## 🧪 Testing Internet Connectivity

### Before NAT Gateway

The private EC2 instance did not have direct internet access.

Running:

```bash
yum -y update
```

failed because the private subnet did not have a route to the internet.

### After NAT Gateway

After configuring the NAT Gateway and updating the private route table:

```bash
0.0.0.0/0 → NAT Gateway
```

the private EC2 instance successfully accessed the internet.

The following command completed successfully:

```bash
yum -y update
```

This verified that the private EC2 instance could initiate outbound internet traffic through the NAT Gateway.

---

## 📸 Screenshots

### 1. VPC Configuration

![VPC](./screenshots/01-vpc.png)

### 2. Public and Private Subnets

![Subnets](./screenshots/02-subnets.png)

### 3. Internet Gateway

![Internet Gateway](./screenshots/03-internet-gateway.png)

### 4. Public Route Table

![Public Route Table](./screenshots/04-public-route-table.png)

### 5. EC2 Instances

![EC2 Instances](./screenshots/05-ec2-instances.png)

### 6. SSH Connection to Private EC2

![SSH](./screenshots/06-ssh-private-instance.png)

### 7. NAT Gateway

![NAT Gateway](./screenshots/07-nat-gateway.png)

### 8. Private Route Table

![Private Route Table](./screenshots/08-private-route-table.png)

### 9. Internet Connectivity Test

![NAT Connectivity](./screenshots/09-nat-connectivity-test.png)

### 10. VPC Resource Map

![VPC Resource Map](./screenshots/10-vpc-resource-map.png)

---

## 🧠 Key Concepts Learned

### NAT Gateway

A NAT Gateway allows resources in a private subnet to initiate outbound connections to the internet without requiring a public IP address.

### Public vs. Private Subnet

A public subnet has a route to an Internet Gateway, while a private subnet does not have a direct route to the Internet Gateway.

### Internet Gateway vs. NAT Gateway

| Internet Gateway                                    | NAT Gateway                                             |
| --------------------------------------------------- | ------------------------------------------------------- |
| Provides internet connectivity for public resources | Provides outbound internet access for private resources |
| Used by public subnets                              | Used by private subnets                                 |
| Direct internet connectivity                        | Outbound connection through NAT                         |
| Does not hide the private resource behind NAT       | Translates private source IP for outbound traffic       |

---

## 💡 Key Takeaways

This lab helped me understand how AWS networking components work together to provide secure internet connectivity.

The most important concept demonstrated was:

```text
Private EC2
     ↓
Private Route Table
     ↓
NAT Gateway
     ↓
Internet Gateway
     ↓
Internet
```

A private EC2 instance can access the internet for activities such as software updates while remaining inaccessible to unsolicited inbound internet connections through the NAT Gateway.

---

## 🚀 Skills Demonstrated

* AWS VPC configuration
* CIDR and subnetting
* Public and private subnet design
* Route table configuration
* Internet Gateway
* NAT Gateway
* Elastic IP
* Amazon EC2
* Linux administration
* SSH
* Network troubleshooting
* AWS cloud networking

---

## 📚 Program

**AWS re/Start Cloud Computing Program**

This lab was completed as part of my hands-on AWS cloud training.

**Status:** ✅ Completed
