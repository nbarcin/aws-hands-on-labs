# AWS CloudFormation: Infrastructure as Code Lab

## Project Overview

This lab demonstrates how to use **AWS CloudFormation** to provision and manage AWS infrastructure using YAML templates.

The project begins by deploying a VPC environment and then progressively updates the CloudFormation stack to add an Amazon S3 bucket and an Amazon EC2 instance. Finally, the stack is deleted to demonstrate infrastructure cleanup.

## Objectives

- Deploy AWS infrastructure using a CloudFormation YAML template.
- Understand the `Parameters`, `Resources`, and `Outputs` sections.
- Create and configure a Virtual Private Cloud (VPC).
- Add an Amazon S3 bucket to an existing CloudFormation stack.
- Deploy an EC2 instance using an AMI retrieved through AWS Systems Manager Parameter Store.
- Reference resources within a template using `!Ref`.
- Update an existing stack and review proposed infrastructure changes.
- Delete the stack and clean up the resources it manages.

## AWS Services Used

- **AWS CloudFormation** — Infrastructure as Code and stack management.
- **Amazon VPC** — Virtual networking.
- **Amazon EC2** — Virtual machine deployment.
- **Amazon S3** — Object storage.
- **AWS Systems Manager Parameter Store** — Retrieves the Amazon Linux 2 AMI ID.
- **Amazon EC2 Security Groups** — Controls inbound network traffic.

## Architecture and Implementation

### Task 1: Deploy a CloudFormation Stack

Created a CloudFormation stack named `Lab` using the provided `task1.yaml` template.

The initial template provisions:
- A VPC with a configurable CIDR range.
- An Internet Gateway attached to the VPC.
- A public subnet.
- A public route table and route to the Internet Gateway.
- A route table association.
- An application security group allowing inbound HTTP traffic on port 80.
- A CloudFormation output exposing the VPC's default security group.

The stack's Events and Resources tabs were used to monitor deployment progress and resource creation.

### Task 2: Add an Amazon S3 Bucket

Modified the existing CloudFormation template by adding an `AWS::S3::Bucket` resource.

```yaml
S3Bucket:
  Type: AWS::S3::Bucket
```

Updated the existing `Lab` stack using the revised template.

CloudFormation created the new bucket while leaving the existing resources unchanged. The bucket name was generated automatically by AWS.

### Task 3: Add an Amazon EC2 Instance

Updated the template to include an AMI parameter and an EC2 instance.

The `AmazonLinuxAMIID` parameter retrieves an Amazon Linux 2 AMI ID through Systems Manager Parameter Store:

```yaml
AmazonLinuxAMIID:
  Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
  Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2
```

The EC2 instance configuration includes:
- **AMI:** Referenced through `!Ref AmazonLinuxAMIID`.
- **Instance type:** `t3.micro`.
- **Security group:** Referenced through `!Ref AppSecurityGroup`.
- **Subnet:** Referenced through `!Ref PublicSubnet`.
- **Name tag:** `App Server`.

This task demonstrates how CloudFormation resources can reference parameters and other resources within the same template.

### Task 4: Delete the CloudFormation Stack

Deleted the `Lab` stack after completing the deployment tasks.

CloudFormation removed the resources managed by the stack, demonstrating how infrastructure can be cleaned up through stack management.

## Key Concepts Learned

- Infrastructure as Code (IaC)
- CloudFormation templates written in YAML
- CloudFormation stack creation and updates
- VPC, subnet, route table, and Internet Gateway configuration
- Security group configuration
- Amazon S3 bucket provisioning
- EC2 instance deployment
- Systems Manager Parameter Store integration
- Resource references using `!Ref`
- Reviewing stack events and change previews
- Infrastructure lifecycle management and resource cleanup

## Repository Structure

```text
aws-cloudformation-infrastructure-as-code-lab/
├── README.md
├── task1.yaml
├── task2.yaml
└── task3.yaml
```

- `task1.yaml` — Initial VPC and networking infrastructure.
- `task2.yaml` — Updated template with an Amazon S3 bucket.
- `task3.yaml` — Updated template with an Amazon S3 bucket and EC2 instance.

*The template files should be added to the repository if you choose to retain the completed versions.*

## Cleanup and Cost Considerations

The lab concludes by deleting the CloudFormation stack to remove its managed resources. Verify that resources have been deleted and check for any remaining billable resources.

## References

- [AWS CloudFormation User Guide](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)
- [CloudFormation Parameters](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html)
- [CloudFormation Resources](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/resources-section-structure.html)
- [Amazon S3 Template Snippets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/quickref-s3.html)
- [AWS::EC2::Instance](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-instance.html)

---

**Skills demonstrated:** AWS CloudFormation, Infrastructure as Code, YAML, Amazon VPC, Amazon EC2, Amazon S3, Systems Manager Parameter Store, and cloud resource lifecycle management.

