
# AWS Lab: Managing Resources with Tagging

## Project Overview

This lab demonstrates how to manage Amazon EC2 resources efficiently using AWS resource tags, the AWS Command Line Interface (CLI), and automation scripts.

Resource tags help organize AWS infrastructure, identify resources by project or environment, automate operational tasks, and enforce security policies.

The lab focuses on filtering EC2 instances by tags, updating resource metadata, stopping and starting instances based on tag values, and identifying non-compliant instances that do not have a required `Environment` tag.

## Objectives

- Apply and update tags on existing AWS resources.
- Find EC2 instances using tag-based filters.
- Use JMESPath expressions with the AWS CLI `--query` option.
- Automate tag updates with Bash scripts.
- Stop and start EC2 instances based on their tags.
- Identify and terminate instances that do not comply with tagging requirements.
- Understand how resource tagging supports automation, governance, and security.

## AWS Services and Tools

- **Amazon EC2:** Virtual servers and instance lifecycle management.
- **Amazon VPC:** Network isolation and subnet configuration.
- **AWS CLI:** Command-line management of AWS resources.
- **AWS SDK for PHP:** Programmatic EC2 resource management.
- **Bash:** Shell scripting and task automation.
- **JMESPath:** Filtering and formatting JSON responses.

## Lab Environment

The lab environment includes:

- One Amazon VPC named `Lab VPC`.
- One public subnet and one private subnet.
- One Linux EC2 instance named `CommandHost`.
- Eight additional Linux EC2 instances.
- AWS CLI tools and preconfigured scripts on the Command Host.

### Resource Tags

| Tag Key | Example Values | Purpose |
|---|---|---|
| `Project` | `ERPSystem`, `Experiment1` | Identifies the associated project |
| `Version` | `1.0`, `1.1` | Tracks the project version |
| `Environment` | `development`, `staging`, `production` | Identifies the deployment environment |

## Tasks Performed

### Task 1: Find and Manage Resources Using Tags

Used the AWS CLI to find EC2 instances associated with the `ERPSystem` project.

**Filter instances by project:**

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Project,Values=ERPSystem"
```

**Return only instance IDs:**

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Project,Values=ERPSystem" \
  --query 'Reservations[*].Instances[*].InstanceId'
```

**Display instance IDs and Availability Zones:**

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Project,Values=ERPSystem" \
  --query 'Reservations[*].Instances[*].{ID:InstanceId,AZ:Placement.AvailabilityZone}'
```

**Find development instances belonging to ERPSystem:**

```bash
aws ec2 describe-instances \
  --filters \
    "Name=tag:Project,Values=ERPSystem" \
    "Name=tag:Environment,Values=development" \
  --query 'Reservations[*].Instances[*].{ID:InstanceId,AZ:Placement.AvailabilityZone}'
```

**Retrieve custom tag values using JMESPath:**

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Project,Values=ERPSystem" \
  --query 'Reservations[*].Instances[*].{ID:InstanceId,Project:Tags[?Key==`Project`]|[0].Value,Environment:Tags[?Key==`Environment`]|[0].Value,Version:Tags[?Key==`Version`]|[0].Value}'
```

These commands demonstrate how AWS CLI filters and JMESPath queries can reduce large JSON responses to useful resource information.

### Task 2: Update Resource Tags with Bash

Reviewed and executed the provided `change-resource-tags.sh` script.

The script identifies development instances belonging to `ERPSystem` and updates their `Version` tag from `1.0` to `1.1`.

**Example script:**

```bash
#!/bin/bash

ids=$(aws ec2 describe-instances \
  --filters \
    "Name=tag:Project,Values=ERPSystem" \
    "Name=tag:Environment,Values=development" \
  --query 'Reservations[*].Instances[*].InstanceId' \
  --output text)

if [ -n "$ids" ]; then
  aws ec2 create-tags \
    --resources $ids \
    --tags Key=Version,Value=1.1
fi
```

Run the script:

```bash
chmod +x change-resource-tags.sh
./change-resource-tags.sh
```

The `create-tags` command creates a tag if it does not exist or updates its value if it already exists.

### Task 3: Stop and Start EC2 Instances by Tag

Examined the provided `stopinator.php` script, which uses the AWS SDK for PHP to identify and manage EC2 instances based on their tags.

**Stop development instances:**

```bash
cd ~/aws-tools

./stopinator.php \
  -t"Project=ERPSystem;Environment=development"
```

**Start the same instances:**

```bash
./stopinator.php \
  -t"Project=ERPSystem;Environment=development" \
  -s
```

The `-t` argument specifies the tag filters, while `-s` instructs the script to start matching instances instead of stopping them.

This approach can help automate development environment schedules and reduce unnecessary compute costs.

### Task 4: Challenge — Terminate Instances Missing Required Tags

The final challenge implements a tag-or-terminate policy.

The objective is to identify EC2 instances in a specified subnet that do not have an `Environment` tag and terminate those non-compliant instances.

The provided `terminate-instances.php` script:

1. Finds instances that have an `Environment` tag.
2. Retrieves all instances in the specified subnet.
3. Compares the two sets of instance IDs.
4. Identifies instances missing the required tag.
5. Terminates the non-compliant instances.

**Example command:**

```bash
./terminate-instances.php \
  -region <region> \
  -subnetid <subnet-id>
```

Replace `<region>` and `<subnet-id>` with the actual values from your lab environment.

**Important:** Termination is destructive. Use this operation only in a disposable lab environment after verifying the subnet and the exact instance IDs. Do not run termination scripts against production resources without appropriate safeguards and authorization.

## Key Concepts Learned

- **Resource tagging:** Organizing cloud resources using key-value pairs.
- **Tag-based filtering:** Finding resources by project and environment.
- **JMESPath queries:** Extracting selected fields from AWS CLI JSON output.
- **Infrastructure automation:** Updating multiple resources with shell scripts.
- **Resource lifecycle management:** Stopping and starting instances programmatically.
- **Cloud governance:** Using tags to support resource organization and policy enforcement.
- **Security compliance:** Identifying resources that fail to meet required tagging rules.

## Security and Best Practices

- Apply consistent tagging standards across AWS resources.
- Require appropriate tags through provisioning workflows and governance controls.
- Follow the principle of least privilege when assigning IAM permissions.
- Validate instance IDs before stopping or terminating resources.
- Use dry-run checks where supported before destructive operations.
- Consider AWS Config and automated remediation for ongoing compliance monitoring.
- Remember that tags alone do not enforce access control or guarantee security.

## Conclusion

This lab provided hands-on experience managing AWS resources through tags, AWS CLI queries, Bash automation, and the AWS SDK for PHP.

The exercises demonstrated how tagging can simplify infrastructure operations, automate resource lifecycle management, and identify resources that do not meet organizational policies.

These skills are useful for cloud administration, DevOps, infrastructure automation, cost optimization, and cloud security governance.

## Skills Demonstrated

`AWS CLI` · `Amazon EC2` · `Amazon VPC` · `Resource Tagging` · `JMESPath` · `Bash Scripting` · `PHP SDK` · `Cloud Automation` · `Security Compliance` · `Resource Lifecycle Management`
