# AWS Lab: EC2 Cost Optimization and Instance Right-Sizing

## Project Overview

This lab demonstrates how to optimize an AWS-hosted Café web application after migrating its local database to Amazon RDS.

The main objective is to reduce infrastructure costs by removing the local MariaDB database server from the Café EC2 instance and downsizing the instance from `t3.small` to `t3.micro`.

The lab uses the AWS Command Line Interface (AWS CLI) to inspect EC2 resources, manage instance states, modify instance attributes, and verify that the web application remains functional after the changes.

## Objectives

- Connect to EC2 instances using SSH.
- Configure the AWS CLI on Linux instances.
- Retrieve EC2 instance information using AWS CLI commands.
- Stop and uninstall the local MariaDB server.
- Identify an EC2 instance using resource tags.
- Stop and resize an EC2 instance.
- Restart the instance and verify its status.
- Validate application functionality after infrastructure changes.
- Understand cost optimization through instance right-sizing and database migration.

## AWS Services and Tools

- **Amazon EC2:** Hosts the Café web application.
- **Amazon RDS:** Provides the managed database service.
- **AWS CLI:** Automates EC2 resource management.
- **Amazon VPC:** Provides the networking environment.
- **Linux and SSH:** Enable remote administration.
- **MariaDB:** Local database server removed during optimization.

## Lab Environment

The lab uses the following components:

- `CafeInstance`: EC2 instance hosting the Café web application.
- `CLI Host`: EC2 instance used to execute AWS CLI commands.
- Amazon RDS database instance: Hosts the migrated application database.
- AWS credentials and Region configuration for CLI access.

## Tasks Performed

### Task 1: Connect to the Café Instance

Connected to the Café EC2 instance through SSH using the provided PEM key.

```bash
chmod 400 labsuser.pem

ssh -i labsuser.pem ec2-user@<public-ip>
```

Replaced `<public-ip>` with the public IPv4 address of the Café instance.

### Task 2: Configure the AWS CLI

Retrieved the AWS Region from the EC2 instance metadata endpoint:

```bash
curl http://169.254.169.254/latest/dynamic/instance-identity/document \
  | grep region
```

Configured the AWS CLI:

```bash
aws configure
```

Provided the lab credentials, default Region, and JSON output format when prompted.

**Security note:** Credentials should be handled securely. Avoid storing long-term access keys in scripts, committing credentials to GitHub, or exposing them in terminal logs. Prefer IAM roles when supported by the lab environment.

### Task 3: Remove the Local MariaDB Server

Stopped the local database service:

```bash
sudo systemctl stop mariadb
```

Uninstalled the MariaDB server package:

```bash
sudo yum -y remove mariadb-server
```

The database had been migrated to Amazon RDS, so the local database server was no longer required for the application architecture.

### Task 4: Identify the Café EC2 Instance

Used the AWS CLI to locate the instance by its `Name` tag:

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=CafeInstance" \
  --query 'Reservations[*].Instances[*].InstanceId' \
  --output text
```

Recorded the returned instance ID for subsequent operations.

### Task 5: Stop and Resize the EC2 Instance

Stopped the Café instance before changing its instance type:

```bash
aws ec2 stop-instances \
  --instance-ids <CafeInstance-Instance-ID>
```

Modified the instance type to `t3.micro`:

```bash
aws ec2 modify-instance-attribute \
  --instance-id <CafeInstance-Instance-ID> \
  --instance-type '{"Value":"t3.micro"}'
```

Starting with a stopped instance allows the instance type to be changed when the requested type is supported by the instance configuration.

### Task 6: Restart and Verify the Instance

Started the downsized instance:

```bash
aws ec2 start-instances \
  --instance-ids <CafeInstance-Instance-ID>
```

Checked the instance type, public DNS name, public IP address, and current state:

```bash
aws ec2 describe-instances \
  --instance-ids <CafeInstance-Instance-ID> \
  --query 'Reservations[*].Instances[*].[InstanceType,PublicDnsName,PublicIpAddress,State.Name]' \
  --output table
```

Repeated the command until the instance reached the `running` state.

**Important:** The public IPv4 address and public DNS name may change after stopping and restarting an instance when it uses an automatically assigned public IP address.

### Task 7: Test the Café Web Application

Opened the following URL in a browser, replacing the hostname with the new public DNS name:

```text
http://<Downsized-CafeInstance-Public-DNS>/cafe
```

Tested the website's available functions to verify that the application remained operational after the local database was removed and the EC2 instance was resized.

## Cost Optimization Strategy

| Optimization | Expected Benefit |
|---|---|
| Remove the local MariaDB server | Reduces local database resource consumption |
| Resize `t3.small` to `t3.micro` | Reduces EC2 compute costs |
| Use Amazon RDS for database hosting | Separates database management from the web server |
| Verify application functionality | Helps ensure cost reductions do not break the application |

Actual savings depend on the AWS Region, pricing model, instance operating system, and usage. Amazon RDS incurs separate charges, so the total architecture cost should be evaluated rather than considering EC2 costs alone.

## Key Concepts Learned

- EC2 instance lifecycle management
- AWS CLI commands and JMESPath queries
- SSH access to Linux instances
- MariaDB server administration
- EC2 instance right-sizing
- Database migration to Amazon RDS
- Public IP and DNS changes after instance restart
- Application validation after infrastructure changes
- Cloud cost optimization

## Best Practices

- Verify the target instance ID before making infrastructure changes.
- Stop an EC2 instance before modifying its instance type.
- Confirm that the database migration is complete before removing the local database server.
- Validate database connectivity and application functionality after migration.
- Use IAM roles and least-privilege permissions wherever possible.
- Monitor application performance after downsizing.
- Review EC2 and RDS costs together to evaluate overall savings.
- Never commit AWS access keys, private keys, or other secrets to a public repository.

## Conclusion

This lab provided hands-on experience optimizing an AWS-hosted web application using the AWS CLI.

By removing an unnecessary local database server, resizing the EC2 instance, and verifying application functionality, the exercise demonstrated how infrastructure right-sizing and managed database services can support cost-efficient cloud architectures.

These skills are relevant to AWS Cloud Administration, DevOps, Infrastructure Automation, and Cloud Cost Optimization.

## Skills Demonstrated

`Amazon EC2` · `Amazon RDS` · `AWS CLI` · `Linux` · `SSH` · `MariaDB` · `Instance Right-Sizing` · `Cloud Cost Optimization` · `Database Migration` · `Infrastructure Management`

