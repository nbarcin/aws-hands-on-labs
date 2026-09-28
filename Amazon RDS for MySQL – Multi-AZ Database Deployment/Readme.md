# Amazon RDS for MySQL – Multi-AZ Database Deployment

## 📌 Lab Overview

In this lab, I deployed a highly available **Amazon RDS for MySQL database** in a custom VPC environment.

The lab demonstrates how to:

* Create a dedicated security group for an RDS database
* Restrict MySQL traffic to a specific EC2 Web Security Group
* Create an RDS DB Subnet Group using private subnets in multiple Availability Zones
* Deploy a **Multi-AZ Amazon RDS for MySQL DB instance**
* Configure private database connectivity
* Connect an EC2-hosted web application to Amazon RDS
* Test database persistence through a web application

---

## 🏗️ Architecture

The lab uses the following architecture:

```text
                    AWS Region
                       |
              +--------+--------+
              |                 |
           AZ 1              AZ 2
              |                 |
       Private Subnet 1   Private Subnet 2
        10.0.1.0/24       10.0.3.0/24
              |                 |
              |                 |
        RDS Primary  <---->  RDS Standby
              |
              |
       Multi-AZ RDS MySQL
          lab database
              |
              |
       EC2 Web Application
              |
       Web Security Group
              |
        Port 3306 allowed
              |
       DB Security Group
```

### Key Security Flow

```text
EC2 Instance
     |
     | MySQL / TCP 3306
     v
DB Security Group
     |
     | Allows traffic ONLY
     | from Web Security Group
     v
RDS MySQL
```

---

# 🛠️ Technologies and AWS Services

* Amazon RDS
* MySQL
* Amazon VPC
* Security Groups
* DB Subnet Groups
* Availability Zones
* Amazon EC2
* Private Subnets
* Multi-AZ Deployment

---

# Task 1 – Create the RDS Security Group

I created a dedicated security group named:

**DB Security Group**

### Configuration

| Setting        | Value                                 |
| -------------- | ------------------------------------- |
| Security Group | DB Security Group                     |
| Description    | Permit access from Web Security Group |
| VPC            | Lab VPC                               |
| Protocol       | MySQL/Aurora                          |
| Port           | TCP 3306                              |
| Source         | Web Security Group                    |

The database security group allows inbound MySQL traffic on port **3306 only from instances associated with the Web Security Group**.

This follows the principle of restricting database access to the required application tier rather than allowing unrestricted access.

### Screenshot

![DB Security Group](screenshots/01-db-security-group.png)

---

# Task 2 – Create the DB Subnet Group

I created an RDS DB Subnet Group named:

**DB Subnet Group**

The subnet group contains private subnets located in two Availability Zones.

### Configuration

| Setting     | Value           |
| ----------- | --------------- |
| Name        | DB Subnet Group |
| VPC         | Lab VPC         |
| Subnet 1    | 10.0.1.0/24     |
| Subnet 2    | 10.0.3.0/24     |
| Subnet Type | Private         |

Using subnets in multiple Availability Zones allows Amazon RDS to support the Multi-AZ deployment.

### Screenshot

![DB Subnet Group](screenshots/02-db-subnet-group.png)

---

# Task 3 – Create the Amazon RDS MySQL Database

I created an Amazon RDS MySQL database using a **Multi-AZ DB instance deployment**.

### Database Configuration

| Setting             | Value                     |
| ------------------- | ------------------------- |
| DB Identifier       | `lab-db`                  |
| Engine              | MySQL                     |
| Deployment          | Multi-AZ DB Instance      |
| Number of Instances | 2                         |
| Instance Class      | `db.t3.medium`            |
| Storage             | General Purpose SSD (gp3) |
| Allocated Storage   | 20 GB                     |
| VPC                 | Lab VPC                   |
| DB Subnet Group     | DB Subnet Group           |
| Public Access       | No                        |
| Security Group      | DB Security Group         |
| Initial Database    | `lab`                     |

The database was configured with **no public access**, keeping the database inside the private network.

### Multi-AZ

The Multi-AZ configuration creates:

```text
Primary DB Instance
        |
        | Synchronous Replication
        |
        v
Standby DB Instance
```

The standby instance is located in a different Availability Zone.

### Screenshot

![RDS Configuration](screenshots/03-rds-configuration.png)

---

# Task 4 – Verify RDS Deployment

After deployment, I verified that the database reached an available state.

### Database

```text
DB Identifier: lab-db
Engine: MySQL
Deployment: Multi-AZ
Status: Available
```

### Screenshot

![RDS Database Available](screenshots/04-rds-available.png)

---

# Connectivity and Security

The RDS database was configured with:

* Public access: **No**
* Private DB Subnet Group
* Dedicated DB Security Group
* MySQL port: **3306**
* Access restricted to the Web Security Group

The RDS endpoint was then used by the web application to establish the database connection.

### Screenshot

![RDS Connectivity and Security](screenshots/05-rds-connectivity.png)

> **Security Note:** Sensitive credentials and passwords should never be committed to GitHub.

---

# Multi-AZ Architecture

The database uses a Multi-AZ deployment.

The architecture provides:

* A primary database instance
* A standby database instance
* Different Availability Zones
* Synchronous replication
* Improved availability and durability

The application connects to the RDS database endpoint rather than directly connecting to an individual database instance.

# 📁 Repository Structure

```text
rds-multi-az-mysql-lab/
│
├── README.md
│
└── screenshots/
    ├── 01-db-security-group.png
    ├── 02-db-subnet-group.png
    ├── 03-rds-configuration.png
    ├── 04-rds-available.png
    ├── 05-rds-connectivity.png
    ├── 06-rds-multi-az.png
    ├── 07-web-application-rds.png
    └── 08-address-book.png
```

---

# 🏁 Lab Outcome

Successfully deployed and tested a **Multi-AZ Amazon RDS for MySQL database** inside a custom VPC.

The web application running on EC2 successfully connected to the private RDS database, and database operations were successfully tested through the application.

**Result: Successful deployment and database connectivity.**
