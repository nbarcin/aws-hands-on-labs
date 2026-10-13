# AWS EC2 Auto Scaling and Application Load Balancer Lab

## Project Overview

This hands-on AWS lab demonstrates how to build a scalable web application infrastructure using Amazon EC2, a custom Amazon Machine Image (AMI), an Application Load Balancer, and an EC2 Auto Scaling group.

The project uses the AWS CLI to launch an EC2 instance and create a custom AMI. The AWS Management Console is then used to configure the load balancer, launch template, Auto Scaling group, and target tracking scaling policy.

The goal is to improve application availability, distribute incoming HTTP traffic, and automatically adjust compute capacity based on CPU utilization.

## Objectives

* Launch and configure an EC2 web server using the AWS CLI.
* Create a custom Amazon Machine Image (AMI).
* Configure an Application Load Balancer (ALB).
* Create an EC2 launch template using the custom AMI.
* Deploy an Auto Scaling group across multiple Availability Zones.
* Configure target tracking based on average CPU utilization.
* Verify target health and test automatic scale-out behavior.
* Understand how load balancing and Auto Scaling support application availability and scalability.

## AWS Services Used

* **Amazon EC2:** Runs the web application.
* **Amazon Machine Image (AMI):** Provides a reusable image of the configured web server.
* **AWS CLI:** Launches EC2 instances and creates the custom AMI.
* **Application Load Balancer (ALB):** Distributes incoming HTTP requests across registered instances.
* **EC2 Launch Template:** Defines the AMI, instance type, and security group for new instances.
* **EC2 Auto Scaling:** Maintains the desired number of instances and adjusts capacity based on demand.
* **Amazon CloudWatch:** Monitors CPU utilization and supports scaling decisions.
* **Elastic Load Balancing Health Checks:** Help Auto Scaling identify unhealthy application instances.
* **Amazon VPC:** Provides network isolation through public and private subnets.

## Architecture

The infrastructure includes an Application Load Balancer in public subnets and an Auto Scaling group running EC2 instances in private subnets across two Availability Zones.

```text
                    Internet
                        |
                        v
              Application Load Balancer
                  (Public Subnets)
                        |
                        v
                  Target Group
                   HTTP : 80
                        |
             +----------+----------+
             |                     |
             v                     v
       EC2 Instance 1        EC2 Instance 2
       Private Subnet 1      Private Subnet 2
             |                     |
             +----------+----------+
                        |
                        v
             Auto Scaling Group
             Min: 2 | Desired: 2
                  Max: 4
                        |
                        v
             CloudWatch Monitoring
                        |
                        v
               Target Tracking
             Average CPU: 50%
```

## Implementation Steps

### 1. Connect to the Command Host

* Connect to the preconfigured Command Host instance using EC2 Instance Connect.
* Verify the AWS Region.
* Configure the AWS CLI.
* Inspect the provided `UserData.txt` initialization script.

### 2. Launch and Test the Web Server

Use the AWS CLI to launch a `t3.micro` EC2 instance with the provided AMI, security group, subnet, and user-data script.

Wait until the instance reaches the running state and retrieve its public DNS name.

Open the web application in a browser and verify that the application loads successfully before creating the AMI.

### 3. Create a Custom AMI

Create an AMI named `WebServerAMI` from the configured EC2 instance.

The custom image captures the web server environment so that Auto Scaling can launch additional instances with the same application configuration.

Wait until the AMI becomes available before using it in the launch template.

### 4. Create an Application Load Balancer

Create an Application Load Balancer named `WebServerELB`.

Configuration highlights:

* VPC: Lab VPC
* Subnets: Two public subnets in separate Availability Zones
* Security group: `HTTPAccess`
* Target group: `webserver-app`
* Listener: HTTP on port 80
* Health check path: `/index.php`

The ALB provides a single DNS endpoint and distributes incoming requests across healthy targets.

### 5. Create a Launch Template

Create a launch template named `web-app-launch-template`.

Configuration highlights:

* AMI: `WebServerAMI`
* Instance type: `t3.micro`
* Security group: `HTTPAccess`
* Key pair: Not included in the template for this lab

The launch template defines the configuration used to provision instances in the Auto Scaling group.

### 6. Configure the Auto Scaling Group

Create an Auto Scaling group named `Web App Auto Scaling Group`.

Configuration highlights:

| Setting                    | Value                     |
| -------------------------- | ------------------------- |
| Launch template            | `web-app-launch-template` |
| Availability Zones         | Two                       |
| Subnets                    | Two private subnets       |
| Minimum capacity           | 2                         |
| Desired capacity           | 2                         |
| Maximum capacity           | 4                         |
| Load balancer target group | `webserver-app`           |
| Scaling policy             | Target tracking           |
| Metric                     | Average CPU utilization   |
| Target value               | 50%                       |

Enable Elastic Load Balancing health checks and attach the Auto Scaling group to the existing target group.

The group initially launches two EC2 instances across the selected Availability Zones.

### 7. Verify the Infrastructure

Verify that:

* Both EC2 instances pass their status checks.
* Both instances register with the `webserver-app` target group.
* Both targets eventually reach a `healthy` state.
* The application is accessible through the ALB DNS name.

### 8. Test Automatic Scaling

Open the application through the load balancer DNS name and select **Start Stress**.

The application generates CPU load on the instance handling the request.

Monitor the following AWS resources:

* **EC2 Auto Scaling Groups:** Check the group’s Activity history.
* **EC2 Instances:** Observe newly launched instances.
* **Target Groups:** Verify the health of registered targets.
* **Amazon CloudWatch:** Monitor CPU utilization and scaling-related metrics.

If the scaling policy determines that additional capacity is required, the Auto Scaling group can launch another instance, up to the configured maximum of four instances.

Scaling is not instantaneous. The result depends on metric collection, evaluation periods, policy behavior, and instance initialization time.

## Key Learnings

* Creating EC2 instances and AMIs with the AWS CLI.
* Using custom AMIs to standardize instance deployments.
* Distributing application traffic with an Application Load Balancer.
* Deploying instances across multiple Availability Zones.
* Separating public load-balancing resources from application instances in private subnets.
* Configuring target tracking scaling policies.
* Understanding the relationship between CloudWatch metrics, scaling policies, and Auto Scaling activities.
* Verifying application availability using target group health checks.

## Security Considerations

* Allow inbound HTTP traffic to the ALB through its security group.
* In production, restrict application instance ingress to the ALB security group instead of allowing unrestricted public HTTP access.
* Keep application instances in private subnets when public IP addresses are not required.
* Follow least-privilege principles for IAM permissions.
* Avoid storing AWS credentials, private keys, or other secrets in the repository.
* Review the security group rules and network configuration before deploying similar infrastructure in a production environment.

## Cleanup

To avoid unnecessary AWS charges, remove the resources created during the lab when they are no longer needed.

1. Delete the Auto Scaling group after reducing its desired capacity or selecting the appropriate deletion options.
2. Delete the Application Load Balancer.
3. Delete the target group when it is no longer in use.
4. Delete the launch template if it is no longer needed.
5. Deregister the custom AMI and delete its associated EBS snapshots when safe to do so.
6. Terminate any standalone EC2 instances created for the lab.
7. Review the VPC and related resources before deleting any lab-specific networking components.

Check for remaining resources and charges in the AWS console.

## Conclusion

This lab demonstrates how to build a scalable web application environment using AWS compute, networking, monitoring, and load-balancing services.

By combining a custom AMI, an Application Load Balancer, and an EC2 Auto Scaling group, the infrastructure can distribute application traffic and adjust compute capacity as demand changes.

The project provides practical experience with AWS CLI automation, high availability, health checks, and dynamic scaling.

---

**Technologies:** AWS EC2 | Amazon AMI | AWS CLI | Application Load Balancer | EC2 Auto Scaling | Amazon CloudWatch | Amazon VPC | Linux | HTTP

