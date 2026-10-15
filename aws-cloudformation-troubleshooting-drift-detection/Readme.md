# AWS CloudFormation Troubleshooting and Drift Detection

## Project Overview

This hands-on AWS lab demonstrates how to deploy, troubleshoot, update, and monitor AWS infrastructure using AWS CloudFormation and the AWS CLI.

The lab focuses on diagnosing a failed CloudFormation stack, investigating EC2 instance initialization logs, correcting a User Data script, detecting configuration drift, and understanding why a stack deletion can fail when an Amazon S3 bucket contains objects.

JMESPath is also used to query JSON-formatted data and filter AWS CLI responses.

## Objectives

- Practice querying JSON data with JMESPath expressions.
- Configure and use the AWS CLI on an EC2 CLI Host instance.
- Deploy AWS resources using CloudFormation templates.
- Troubleshoot failed stack deployments.
- Use CloudFormation failure-handling options to preserve resources for debugging.
- Analyze EC2 `cloud-init` logs and User Data scripts.
- Correct a package installation error in a CloudFormation template.
- Detect configuration drift after manually modifying a security group.
- Understand CloudFormation stack update and deletion behavior.
- Investigate an S3 bucket deletion failure caused by objects remaining in the bucket.

## AWS Services and Tools

- **AWS CloudFormation** — Infrastructure as Code (IaC) and stack lifecycle management.
- **Amazon EC2** — Virtual servers for the CLI Host and Web Server.
- **Amazon VPC** — Virtual networking infrastructure.
- **Amazon S3** — Object storage.
- **AWS CLI** — Command-line management of AWS resources.
- **JMESPath** — JSON querying and filtering.
- **Linux and SSH** — Remote access, command execution, and log analysis.
- **cloud-init** — EC2 instance initialization and User Data execution.

## Lab Tasks

### Task 1: Practice JMESPath Queries

Used the JMESPath website to practice querying JSON documents.

Examples:

Retrieve all dessert records:

```text
desserts
```

Retrieve the second element in an array:

```text
desserts[1]
```

Retrieve an attribute from the first element:

```text
desserts[0].name
```

Retrieve selected attributes:

```text
desserts[0].[name,price]
```

Retrieve the names of all desserts:

```text
desserts[].name
```

Filter records by an attribute value:

```text
desserts[?name=='Carrot cake']
```

Query CloudFormation resources to find the logical ID of an EC2 instance:

```text
StackResources[?ResourceType == 'AWS::EC2::Instance'].LogicalResourceId
```

**Key takeaway:** JMESPath makes it easier to filter and extract specific values from JSON output returned by AWS CLI commands.

### Task 2: Troubleshoot a CloudFormation Stack

#### 2.1 Connect to the CLI Host

Connected to the preconfigured CLI Host EC2 instance using SSH.

#### 2.2 Configure the AWS CLI

Configured the AWS CLI with the lab-provided credentials and the correct AWS Region.

Retrieved the Region from EC2 instance metadata:

```bash
curl http://169.254.169.254/latest/dynamic/instance-identity/document | grep region
```

Configured the CLI:

```bash
aws configure
```

*Security note: Lab credentials and private keys should never be committed to a public GitHub repository.*

#### 2.3 Attempt the Initial Stack Deployment

Inspected the CloudFormation template:

```bash
less template1.yaml
```

Attempted to create the stack:

```bash
aws cloudformation create-stack \
  --stack-name myStack \
  --template-body file://template1.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameters ParameterKey=KeyName,ParameterValue=vockey
```

Monitored resource creation:

```bash
aws cloudformation describe-stack-resources \
  --stack-name myStack \
  --query 'StackResources[*].[ResourceType,ResourceStatus]' \
  --output table
```

Investigated failed stack events:

```bash
aws cloudformation describe-stack-events \
  --stack-name myStack \
  --query "StackEvents[?ResourceStatus == 'CREATE_FAILED']"
```

**Finding:** The `WaitCondition` timed out because the EC2 User Data script failed. CloudFormation rolled back the stack, deleting the EC2 instance before its logs could be inspected.

#### 2.4 Preserve Resources for Troubleshooting

Recreated the stack with rollback disabled on failure:

```bash
aws cloudformation create-stack \
  --stack-name myStack \
  --template-body file://template1.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --on-failure DO_NOTHING \
  --parameters ParameterKey=KeyName,ParameterValue=vockey
```

Connected to the Web Server EC2 instance and inspected the initialization logs:

```bash
tail -50 /var/log/cloud-init-output.log
```

Inspected the executed User Data script:

```bash
sudo cat /var/lib/cloud/instance/scripts/part-001
```

**Root cause:** The User Data script attempted to install a package named `http`, which was unavailable. The correct Apache web server package name for this lab's Amazon Linux environment was `httpd`.

Because the script used `#!/bin/bash -e`, execution stopped when the package installation command failed. Consequently, the WaitCondition never received its success signal.

#### 2.5 Fix the Template and Deploy Successfully

Edited the CloudFormation template:

```bash
vim template1.yaml
```

Changed the package installation command from `http` to `httpd`.

Verified the change:

```bash
cat template1.yaml | grep httpd
```

Deleted the failed stack, waited for its deletion to complete, and redeployed the corrected template.

Verified the final stack status:

```bash
aws cloudformation describe-stacks \
  --stack-name myStack \
  --output table
```

**Expected result:** `CREATE_COMPLETE`

Tested the Web Server using the public IP address provided in the stack outputs.

### Task 3: Modify Resources and Detect Drift

#### 3.1 Manually Modify the Security Group

Updated the inbound SSH rule of the `WebServerSG` security group through the AWS Management Console.

Changed the SSH source from `0.0.0.0/0` to `My IP`, restricting SSH access to the current public IP address detected by AWS.

#### 3.2 Upload an Object to Amazon S3

Retrieved the S3 bucket name from the CloudFormation stack outputs:

```bash
bucketName=$(aws cloudformation describe-stacks \
  --stack-name myStack \
  --query "Stacks[*].Outputs[?OutputKey == 'BucketName'].[OutputValue]" \
  --output text)

echo "bucketName = $bucketName"
```

Created an empty file:

```bash
touch myfile
```

Uploaded the file:

```bash
aws s3 cp myfile s3://$bucketName/
```

Verified the object:

```bash
aws s3 ls "$bucketName/"
```

#### 3.3 Detect Configuration Drift

Started drift detection:

```bash
aws cloudformation detect-stack-drift \
  --stack-name myStack
```

Checked the drift detection status using the returned detection ID:

```bash
aws cloudformation describe-stack-drift-detection-status \
  --stack-drift-detection-id <drift-detection-id>
```

Reviewed resource status and drift information:

```bash
aws cloudformation describe-stack-resources \
  --stack-name myStack \
  --query 'StackResources[*].[ResourceType,ResourceStatus,DriftInformation.StackResourceDriftStatus]' \
  --output table
```

Retrieved details of modified resources:

```bash
aws cloudformation describe-stack-resource-drifts \
  --stack-name myStack \
  --stack-resource-drift-status-filters MODIFIED
```

**Findings:**

- The manually modified security group was detected as `MODIFIED`.
- The SSH rule no longer matched the configuration defined in the original template.
- Adding an object to the S3 bucket did not register as drift because it did not change a property tracked by CloudFormation drift detection.

Attempted to update the stack using the original template:

```bash
aws cloudformation update-stack \
  --stack-name myStack \
  --template-body file://template1.yaml \
  --parameters ParameterKey=KeyName,ParameterValue=vockey
```

The lab demonstrates that detecting drift and resolving drift are separate tasks. An update using the original template is not guaranteed to succeed or automatically reconcile every difference.

### Task 4: Investigate Stack Deletion Failure

Attempted to delete the stack:

```bash
aws cloudformation delete-stack \
  --stack-name myStack
```

Monitored resource deletion:

```bash
aws cloudformation describe-stack-resources \
  --stack-name myStack \
  --query 'StackResources[*].[ResourceType,ResourceStatus]' \
  --output table
```

**Finding:** The S3 bucket failed to delete because it still contained the uploaded object.

CloudFormation does not automatically empty a non-empty S3 bucket when deleting a stack. The remaining objects must be handled before the bucket can be deleted.

## Key Learnings

- **Infrastructure as Code:** CloudFormation templates define and manage AWS resources consistently.
- **Stack rollback:** Default rollback behavior can remove resources needed for troubleshooting.
- **Failure handling:** `--on-failure DO_NOTHING` preserves resources after a failed stack creation, allowing investigation.
- **EC2 troubleshooting:** `cloud-init-output.log` helps identify startup and User Data errors.
- **Shell scripting:** The `-e` option stops a Bash script when a command fails.
- **WaitCondition:** A WaitCondition depends on a success signal; a missing signal can cause a timeout and stack failure.
- **JMESPath:** The `--query` parameter filters AWS CLI JSON output into more useful results.
- **Configuration drift:** Manual changes can cause deployed resources to differ from their CloudFormation templates.
- **S3 deletion:** A bucket containing objects may prevent successful stack deletion.
- **Security best practices:** SSH access should be restricted to trusted source IP addresses rather than opened to the entire internet.

## Conclusion

This lab provided practical experience with AWS CloudFormation deployment, failure analysis, template correction, and configuration drift detection. By identifying and correcting an invalid package name in the EC2 User Data script, I successfully deployed the stack and verified the web server.

The exercise also demonstrated why infrastructure changes should be managed through CloudFormation templates, why drift detection matters, and why resource dependencies and stored objects must be considered during stack deletion.

