+++
title = 'Cloud'
date = '2025-09-07T19:08:16-04:00'
weight = 10
draft = false
+++


Cloud providers manage the physical hardware, so you do not need to monitor components or replace failed devices. All physical servers eventually fail, and cloud infrastructure abstracts that risk away.

Virtual private server (VPS) providers offer virtual machines at lower cost but with fewer features than full cloud platforms like AWS or GCP. Common VPS providers include DigitalOcean, Linode, and Amazon Lightsail.

Even cloud services experience outages, so automation is critical. Automated deployments let you redeploy quickly when problems occur. Back up your cloud resources regularly, keep copies of server images, and store at least one recent backup outside your primary cloud infrastructure.

## AWS

AWS treats servers as disposable rather than permanent. Two features support this approach:

- **Auto scaling**: scales resources up as traffic increases, with configurable limits on how far it can scale.
- **Auto healing**: disposes of an instance that fails a health check and replaces it with a new one.

Automate server rebuilds so you can recover quickly from failures.

An **IAM role** lets you assign permissions to a role and attach that role to an AWS resource, such as an EC2 instance. **Tags** are key/value pairs you attach to instances to organize and identify resources.

### Services overview

| Service | Description |
|:---|:---|
| Virtual Private Cloud (VPC) | A software-defined network with routers, gateways, firewalls, and virtual machines. |
| Elastic Compute Cloud (EC2) | Runs virtual machines. Each VM is an EC2 instance. |
| Elastic Block Storage (EBS) | Block storage equivalent to a hard disk volume. |
| Elastic Load Balancer (ELB) | Routes traffic across multiple EC2 instances running your application. |
| Identity and Access Management (IAM) | Creates and manages user accounts, permissions, and API keys for programmatic access. |
| Route 53 | Manages DNS records and domain name registration. |
| Simple Storage Service (S3) | Object storage. Objects are name/value pairs stored in buckets. Each bucket name is globally unique. Unlike block storage, S3 has no filesystem and does not enforce Unix permissions. |
| Elastic Kubernetes Service (EKS) | Runs Kubernetes on AWS without managing the control plane yourself. |
| Security groups | Controls access to AWS resources by IP address and port. Most AWS resources are inaccessible from the public internet by default. Security groups define which IPs and ports are allowed. |

### Create admin account

When you first log in, only the root account exists. Create a dedicated admin IAM user immediately and set up multi-factor authentication (MFA) before using the account. Authy is a good choice for MFA:

1. Go to **IAM**.
2. Select **Users** in the left pane.
3. Select **Create user**.
4. Enable console access.
5. For permissions, select **Attach Policies Directly**.
6. In **Permissions policies**, select **AdministratorAccess**.
7. Skip tags and select **Create user**.

### Choosing a region

Each AWS account includes one VPC per region. You can create additional VPCs if needed. Deploy instances in the region closest to your users to minimize latency.

AWS offers two levels of geographic subdivision within a region:

- **Availability zones**: Isolated data centers within a region that reduce the impact of localized failures.
- **Local zones**: Infrastructure placed closer to specific cities, reducing latency further for users in those areas.

**CloudFront** is AWS's content delivery network (CDN). It caches content at edge locations worldwide, serving requests from the location nearest to the user.

### Creating a role

Session Manager is part of AWS Systems Manager. It gives you shell access to an EC2 instance without managing SSH keys or opening port 22. You control access through the AWS console. Session Manager requires the SSM agent installed on the instance and an IAM role with the appropriate permissions.

To create the role:

1. Go to **IAM** > **Roles**.
2. Select **Create Role**.
3. Select **AWS Service** and **EC2** as the use case.
4. Search for and attach the **AmazonSSMFullAccess** policy.
5. Enter a name for the role.
6. Select **Create Role**.

### Deploying an EC2 instance

To launch a new EC2 instance:

1. Search for **EC2** and select **Instances** from the left pane.
2. Select **Launch Instance**.
3. Enter a name and select your distribution.
4. In **Instance type**, select **t2.micro** (free tier eligible).
5. In **Key pair**, select **Create new key pair**. Enter a name and accept the defaults. Store the downloaded key in a safe place.
6. In **Network settings**, set the source to **Anywhere** (`0.0.0.0`) to allow access from any IP, or select **My IP** to restrict access to your current address. If your IP is not static, you will need to update the security group each time your IP changes.
7. Select the checkboxes to allow HTTPS and HTTP traffic if you are deploying a web server.
8. In **Configure storage**, set the root volume size. 30 GiB is sufficient for most test environments.
9. In **Advanced details**, add any commands to run at instance creation in the **User data** field:
   ```bash
   #!/bin/bash
   apt update
   apt dist-upgrade -y
   apt install -y apache2
   ```
10. Select **Launch instance**.

After the instance starts, the Apache2 default page should be accessible at the instance's public IP address.

### Accessing an EC2 instance

Attach the Session Manager IAM role to the instance before connecting. You may need to reboot the instance after attaching the role for it to take effect:

1. Go to **EC2** > **Instances**.
2. Select the instance row, then select **Actions** > **Security** > **Modify IAM role**. Select the IAM role with Session Manager access.
3. Select the **Instance ID** of the instance you want to connect to.
4. Select **Connect**.
5. Select the **Session Manager** tab, then select **Connect**.
   

### Create an AMI

An Amazon Machine Image (AMI) is a disk image of an instance. Use an AMI to replicate a configured server or recover quickly from a failure. Shut down the instance before creating an AMI to prevent writes from corrupting the image:

1. Go to **EC2** > **Instances**, right-click the instance, and select **Image and templates** > **Create image**.
2. Enter a name and description.
3. Accept the remaining defaults and select **Create image**.
4. Select the AMI ID in the notification, or go to **Images** > **AMIs** to view your AMI.
5. When **Status** is **Ready**, right-click the AMI and select **Launch instance from AMI**.

### Auto Scaling

Auto Scaling brings additional instances online automatically as traffic increases. It requires a launch template, which captures the configuration choices you would otherwise make manually each time you create an instance.

#### Create a launch template

A launch template defines the instance configuration Auto Scaling uses when it provisions new servers:

1. Go to **EC2** > **Instances** > **Launch Templates**.
2. Select **Create launch template**.
3. Configure the following settings:
   - **Launch template name** and **Template version description**
   - **Application and OS Images (AMI)**: select **My AMIs** > **Owned by me**, then select your AMI
   - **Instance type**: t2.micro
   - **Key pair (login)**: select your `.pem` key pair
   - **Network settings**: select an existing security group
   - **Advanced details** > **IAM instance profile**: select your profile
4. Select **Create launch template**.

#### Create an Auto Scaling group

An Auto Scaling group is a logical collection of instances that serve a common application. A target group is a subset of those instances that a load balancer routes traffic to:

1. Go to **EC2** > **Auto Scaling Groups**.
2. Select **Create Auto Scaling Group**.
3. Enter a name and select your launch template.
4. In **Network**, accept the default VPC and select the first availability zone.
5. In **Load balancing**, select **Attach to a new load balancer**.
6. Configure the load balancer:
   - Select **Application Load Balancer**.
   - Enter a **Load balancer name**.
   - Select two options under **Availability Zones and subnets**.
   - Under **Default routing (forward to)**, select **Create a target group** and enter a name.
   - Select **Next**.
7. On **Configure group size and scaling**, set the minimum and maximum instance counts. Set both to **1** for a test environment.
8. Select **Skip to review**, then select **Create Auto Scaling group**.

When the group is created, it provisions one instance to meet the minimum capacity. Access the application at the DNS name shown under **Load Balancing** > **Load Balancers**.

### Costs

Monitor your AWS expenses in the **Billing and Cost Management** dashboard. Access it by searching for **Billing** in the console. Manage billing with the root account.

To keep costs under control:

- Set up a billing alert to notify you when spending exceeds a threshold.
- Delete backups you no longer need.
- Stop or terminate EC2 instances when they are not in use.

### Commands

Connect to an EC2 instance using your `.pem` key pair:

```bash
ssh -i /path/to/ec2-key.pem ubuntu@<public-instance-ip>         # connect to an EC2 instance
```