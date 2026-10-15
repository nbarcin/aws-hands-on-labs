
# AWS S3 Secure File Sharing and Event Notifications

## Project Overview

This hands-on lab demonstrates how to configure a secure Amazon S3 bucket for file sharing, manage access using AWS Identity and Access Management (IAM), and automate email notifications using Amazon Simple Notification Service (SNS).

The lab simulates a media company that needs to securely share image files with an external user. The user should be able to view, upload, and delete images within a specific S3 prefix without having permission to modify bucket-level settings.

Amazon S3 event notifications are configured to send email alerts whenever objects are created or deleted within the designated `images/` prefix.

## Objectives

- Connect to an Amazon EC2 instance using EC2 Instance Connect.
- Configure the AWS CLI.
- Create and manage an Amazon S3 bucket.
- Upload and synchronize image files using AWS CLI commands.
- Review IAM groups, users, and permission policies.
- Apply least-privilege access to an S3 bucket prefix.
- Test authorized and unauthorized operations.
- Create an SNS topic and subscribe an email endpoint.
- Configure Amazon S3 event notifications.
- Verify email notifications for object creation and deletion.

## AWS Services and Tools

- **Amazon S3:** Object storage and secure file sharing.
- **Amazon EC2:** CLI Host used to execute AWS CLI commands.
- **AWS IAM:** User, group, and policy-based access control.
- **Amazon SNS:** Email notifications for S3 events.
- **AWS CLI:** Command-line management of AWS resources.
- **JSON:** IAM policies and S3 event notification configuration.

## Architecture Overview

The solution uses the following workflow:

1. An administrator creates an S3 bucket and uploads image files.
2. An IAM group named `mediaco` provides permissions to the `mediacouser` IAM user.
3. The external user can view, upload, and delete objects within the `images/` prefix.
4. The external user cannot modify bucket permissions.
5. Amazon S3 detects object creation and deletion events.
6. S3 publishes event notifications to an SNS topic.
7. SNS sends email notifications to confirmed subscribers.

### Architecture Diagram

```text
Administrator
     |
     v
AWS CLI on EC2 CLI Host
     |
     v
Amazon S3 Bucket
     |
     +---- images/
     |       |
     |       +---- Upload objects
     |       +---- View objects
     |       +---- Delete objects
     |
     +---- IAM Access Control
     |       |
     |       +---- mediaco IAM Group
     |                |
     |                +---- mediacouser
     |
     +---- S3 Event Notifications
                  |
                  v
             Amazon SNS Topic
             s3NotificationTopic
                  |
                  v
             Email Subscriber
```

## Task 1: Connect to the CLI Host and Configure AWS CLI

### Connect to EC2

1. Open the AWS Management Console.
2. Navigate to **EC2 → Instances**.
3. Select the `CLI Host` instance.
4. Choose **Connect**.
5. Open the **EC2 Instance Connect** tab and choose **Connect**.

### Configure AWS CLI

Run:

```bash
aws configure
```

Enter the required credentials and configuration:

- AWS Access Key ID: Your authorized access key
- AWS Secret Access Key: Your authorized secret key
- Default region: `us-west-2`
- Default output format: `json`

**Security note:** Use temporary credentials or IAM roles whenever possible. Do not commit access keys or secret keys to GitHub.

## Task 2: Create and Initialize the S3 Bucket

Create a bucket with a globally unique name beginning with `cafe-`.

```bash
aws s3 mb s3://cafe-xxxnnn --region us-west-2
```

Replace `cafe-xxxnnn` with your own lowercase, globally unique bucket name.

### Upload Image Files

Synchronize the initial image directory with the S3 bucket:

```bash
aws s3 sync ~/initial-images/ s3://cafe-xxxnnn/images
```

### Verify Uploaded Objects

```bash
aws s3 ls s3://cafe-xxxnnn/images/ --human-readable --summarize
```

This command lists the uploaded objects and summarizes their total size and count.

## Task 3: Review IAM Permissions

### Review the IAM Group

1. Navigate to **IAM → User groups**.
2. Select the `mediaco` group.
3. Open the **Permissions** tab.
4. Review the `IAMUserChangePassword` and `mediaCoPolicy` policies.

The `mediaCoPolicy` includes permissions for listing buckets and restricting object-level operations to the designated S3 prefix.

The primary object permissions are:

- `s3:GetObject`: Read objects.
- `s3:PutObject`: Upload objects.
- `s3:DeleteObject`: Delete objects.

### Review the IAM User

1. Navigate to **IAM → Users**.
2. Select `mediacouser`.
3. Review the **Permissions** and **Groups** tabs.
4. Confirm that the user inherits permissions from the `mediaco` group.

The lab also creates an access key to test CLI access. In a production environment, prefer temporary credentials and avoid long-lived IAM user access keys when possible.

### Test Access Permissions

Sign in as `mediacouser` in a separate browser or private browsing session.

Test the following operations:

| Operation | Expected result |
|---|---|
| View an image under `images/` | Allowed |
| Upload an image under `images/` | Allowed |
| Delete an image under `images/` | Allowed |
| Modify bucket permissions | Denied |
| Change an object's ACL to public-read | Denied |

This demonstrates the principle of **least privilege** by allowing only the required actions.

## Task 4: Configure Amazon SNS Notifications

### Create an SNS Topic

1. Navigate to **Amazon SNS → Topics**.
2. Choose **Create topic**.
3. Select **Standard**.
4. Enter the topic name:

```text
s3NotificationTopic
```

5. Create the topic.
6. Copy its **Topic ARN** for use in the following configuration steps.

### Configure the SNS Access Policy

Update the topic access policy so that Amazon S3 can publish messages to the SNS topic.

Replace the placeholders with your actual SNS topic ARN and S3 bucket name.

```json
{
  "Version": "2008-10-17",
  "Id": "S3PublishPolicy",
  "Statement": [
    {
      "Sid": "AllowPublishFromS3",
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": "SNS:Publish",
      "Resource": "arn:aws:sns:us-west-2:ACCOUNT_ID:s3NotificationTopic",
      "Condition": {
        "ArnLike": {
          "aws:SourceArn": "arn:aws:s3:::cafe-xxxnnn"
        }
      }
    }
  ]
}
```

Replace `ACCOUNT_ID` with the AWS account ID in the SNS topic ARN. Replace `cafe-xxxnnn` with your actual bucket name.

This policy allows the specified S3 bucket to publish messages to the SNS topic.

### Subscribe an Email Address

1. Open `s3NotificationTopic`.
2. Choose **Create subscription**.
3. Select the topic ARN.
4. Set the protocol to `Email`.
5. Enter an email address you can access.
6. Create the subscription.
7. Open the confirmation email and choose **Confirm subscription**.

The email subscription must be confirmed before SNS can deliver notifications to that endpoint.

## Task 5: Configure S3 Event Notifications

Create a file named `s3EventNotification.json` on the CLI Host.

```bash
vi s3EventNotification.json
```

Add the following configuration, replacing the topic ARN with your actual SNS topic ARN.

```json
{
  "TopicConfigurations": [
    {
      "TopicArn": "arn:aws:sns:us-west-2:ACCOUNT_ID:s3NotificationTopic",
      "Events": [
        "s3:ObjectCreated:*",
        "s3:ObjectRemoved:*"
      ],
      "Filter": {
        "Key": {
          "FilterRules": [
            {
              "Name": "prefix",
              "Value": "images/"
            }
          ]
        }
      }
    }
  ]
}
```

Save the file and exit the editor.

Apply the notification configuration:

```bash
aws s3api put-bucket-notification-configuration \
  --bucket cafe-xxxnnn \
  --notification-configuration file://s3EventNotification.json
```

Replace the bucket name and topic ARN with your actual values.

This configuration requests notifications for object creation and removal events involving objects under the `images/` prefix.

## Task 6: Test S3 Event Notifications

### Upload an Object

Upload a sample image:

```bash
aws s3api put-object \
  --bucket cafe-xxxnnn \
  --key images/Caramel-Delight.jpg \
  --body ~/new-images/Caramel-Delight.jpg
```

Expected result: An S3 object creation event is published to SNS, and a notification email is delivered to the confirmed subscriber.

### Download an Object

```bash
aws s3api get-object \
  --bucket cafe-xxxnnn \
  --key images/Donuts.jpg \
  Donuts.jpg
```

A download does not trigger the configured notifications because the configuration listens only for object creation and removal events.

### Delete an Object

```bash
aws s3api delete-object \
  --bucket cafe-xxxnnn \
  --key images/Strawberry-Tarts.jpg
```

Expected result: An object removal event is published to SNS, and a notification email is delivered.

### Test an Unauthorized ACL Operation

Run:

```bash
aws s3api put-object-acl \
  --bucket cafe-xxxnnn \
  --key images/Donuts.jpg \
  --acl public-read
```

Expected result: The operation fails with `AccessDenied` when the user does not have the required permission.

This confirms that the user cannot make the object publicly readable through this operation.

## Troubleshooting

### S3 Bucket Creation Fails

- Verify that the bucket name is globally unique.
- Use lowercase letters and numbers where possible.
- Confirm that the selected AWS Region is correct.

### Access Denied Errors

- Review the IAM user's group membership.
- Inspect the attached IAM policies.
- Verify that the requested operation targets the permitted `images/` prefix.
- Check for explicit denies and applicable bucket policies.

### SNS Email Notifications Are Not Received

- Confirm the email subscription.
- Verify the SNS topic ARN in the S3 notification configuration.
- Ensure that the SNS topic policy permits publishing from the correct S3 bucket.
- Verify that the bucket and SNS topic are in compatible Regions.
- Confirm that the operation is an object creation or removal event under `images/`.

### Event Configuration Fails

- Validate the JSON syntax.
- Confirm that the topic ARN is correct.
- Ensure that the SNS topic access policy permits the S3 service to publish notifications.

## Security Best Practices

- Apply least-privilege IAM policies.
- Restrict user permissions to the required bucket and prefix.
- Keep S3 Block Public Access enabled unless public access is explicitly required.
- Do not store AWS credentials in source code or public repositories.
- Prefer IAM roles and temporary credentials for AWS CLI access.
- Use encryption at rest and in transit.
- Monitor relevant activity using AWS CloudTrail.
- Avoid using public-read ACLs for private business data.


## Key Takeaways

- Amazon S3 provides scalable object storage.
- IAM groups simplify permission management for multiple users.
- Prefix-scoped permissions help limit access to specific objects.
- Least privilege reduces unnecessary access.
- Amazon SNS can deliver email notifications to confirmed subscribers.
- S3 event notifications can respond to object creation and removal.
- IAM and SNS resource policies work together to authorize event publishing.
- AWS CLI commands can automate common storage and notification tasks.

## Conclusion

This lab demonstrates how to build a secure file-sharing workflow using Amazon S3, IAM, AWS CLI, and Amazon SNS. It combines access control, event-driven notifications, and practical security testing to illustrate how AWS services can support external file-sharing requirements while limiting unauthorized operations.

## Skills Demonstrated

`Amazon S3` · `AWS IAM` · `AWS CLI` · `Amazon SNS` · `JSON Policies` · `Least Privilege` · `Event Notifications` · `Cloud Security` · `Access Control` · `Troubleshooting`

## Repository

**Suggested repository name:** `aws-s3-secure-file-sharing-event-notifications`
