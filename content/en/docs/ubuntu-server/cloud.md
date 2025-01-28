---
title: "Deploying Ubuntu in the Cloud"
linkTitle: "Cloud"
weight: 170
# description:
---


- do not need to worry about monitoring hardware components or replacing failed physical devices
  - all physical servers will fail eventually
- VPS - virtual private server providers - provide VMs but usually don't provide as many features as a cloud solution like AWS or GCP
  - DigitalOcean
  - Linode
  - Amazon Lightsail
- Even cloud services have outages
- Automation is key in the cloud, helpful with redeploys
- You still have to backup cloud resources
  - Backup images and servers
  - Keep copy of most recent backup outside cloud infra

## AWS

Treats servers as disposable, not like pets:
- Auto scaling - you can configure resources to scale as traffic increases, but alsos set limits on how much it increases by
- Auto healing - instance is disposed of and replaced with a new one
  - When server fails a health check
- Always consider automation for rebuilding servers in case of a problem
- IAM role lets you assign permissions to a role, and then attach that role to an AWS resource, like an EC2 instance
- Tags are key/value pairs that you can make up and attach to any instances

### Services overview

Brief description of some core services:

Virtual Private Cloud (VPC)
: Software version of a complete network, with routers, gateways, firewalls, VMs, etc

Elastic Compute Cloud (EC2)
: Runs VMs, similar to a level 1 hypervisor. Each VM is an _EC2 instance_.

Elastic Block Storage (EBS)
: Block storage, the same as a hard disk volume.

Elastic Load Balancer (ELB)
: Routes traffic between multiple EC2 instances that run your app

Identify and Access Management (IAM)
: User privilege management. Tool to create and manage user accts, determine user permissions, create API keys for programatic access

Route 53
: Helps manage DNS entries and registering domain names.

Simple Storage Service (S3)
: Object storage, which is different from physical disk that you add to server, partition, format, and mount.
  - Object storage does not have a filesystem and doesn't understand permissions.
  - Each object is a name/object pair.
  - Create buckets that store the objects. Each bucket name is unique

Elastic Kubernetes Service (EKS)
: Runs K8s on AWS. Much simpler than deploying your own resources.

Security groups
: Determines access to AWS resources:
  - Much of AWS is inaccessible from the public internet. For example, EC2 instances can reach the internet, but all ports are blocked for inbound traffic.
  - Can disallow by IP or port
  - Can create a security group that lets specific IPs access an instance

### Create admin account

To begin, you only have the root user account, so log in as root and then create IAM users later:
- Always set up MFA - Authy is a good choice

Create new admin acct:
1. Go to IAM
2. Select Users in left pane
3. Select Create user
4. Give Console access
5. For permissions, select **Attach Policies Directly**
6. In Permissions policies, select **AdministratorAccess**.
7. Skip tags
8. 

### Choosing a region

In your account, you have 1 VPC in each region:
- creating additional VPCs is optional if you ever need more than once
- Create instances as close to the user as possible
- CloudFront is AWS's CDN for resources as close as possible to specified regions
- Availability zones are within regions, and let you get closer to customers
  - Also local zones within availability zones

### Creating a role

Session Manager is part of System Manager, and it lets us access a shell on the EC2 instance:
- alternative to OpenSSH
- added benefit that we do not have to manage the backend security
- can control access to Session Manager through the console
- Requires you install the Session Manager package on the instance, and it requires you create an IAM role with permissions to use Session Manager

Create the role:
1. Go to **IAM** > **Roles**
2. Select **Create Role**
3. Select **AWS Service** and **EC2** as the use case
4. Search for **AmazonSSMFullAccess** policy to the role
5. Name the role
6. Select **Create Role**

### Deploying an EC2 instance

1. Type EC2 in the search bar.
2. Select **Instances** from the left pane. Any instances that you launched are here.
3. Select **Launch Instance**.
4. Add a name and select your distro.
5. In **Instance type**, make sure that **t2.micro** is selected - that is on the free tier.
6. In **Key pair**, select **Create new key pair**.
7. In the popup, enter a name and accept the defaults. This downloads a key to your workstation. Store it in a safe place.
8. In **Network settings**, leave **Anywhere** as `0.0.0.0` if you want to be able to access the instance from any IP. Otherwise, select **My IP** and it will add your public IP.
   Be careful if you are not using a static ip, or you have to manually update the security group for the instance to include your new IP.
9. Because we are setting up a web server, select the boxes to allow HTTPS and HTTP traffic from the internet. Usually, you wouldn't allow this.
10. In **Configure storage**, select how much disk you want on your root volume. You can select 30GIB.
11. In **Advanced details**, enter commands that you want to run when the instance is created in **User data**:
    ```bash
    #!/bin/bash
    apt update
    apt dist-upgrade -y
    apt install -y apache2
    ```
12. Select **Launch instance**.

You should be able to see the Apache2 Default page on the public IP for your instance.

### Accessing an EC2

> Might have to reboot before running this?

Make sure you have an IAM role with SessionManager ready:
1. Go to **EC2** > **Instances** to view and access the running instance.
2. Select the box to the left of the **Name** column in the instance row, then select **Actions** > **Security** > **Modify IAM role**. Select the IAM role with Session Manager access.
3. On **Instances**, select the **Instance ID** for the instance that you want to connect to.
4. Select **Connect**.
5. Select the **Session Manager** tab, then select **Connect**.
   

### Create an AMI

An Amazon Machine Image is a disk image - a copy of the instance's hard disk so you can replicate the original server:
- should shut down server so there are no writes happening, resulting in data corruption

1. Go to **EC2** > **Instances**, and right-click the Instance ID. Select **Image and templates** > **Create image**.
2. Enter a name and description.
3. Accept the other defaults, and select **Create image**.
4. Select the AMI ID in the notificaiton, or go to **Images** > **AMIs** to view your AMI.
5. When **Status** is **Ready**, right-click the instance and select **Launch instance from AMI** to launch a new instance.

### Auto Scaling

Automate the process of bringing more servers online to handle an increase in traffic:
- Create a launch template, which automates all the selections we make when we create an instance

First, you have to create a Launch template:
1. Go to **EC2** > **Instances** > **Launch Templates**.
2. Select **Create launch template**.
3. Make the following selections:
   - Launch template name
   - Template version description
   - Application and OS Images (AMI) > My AMIs > Owned by me > select your AMI
   - Instance type: t2.micro (cost-effective)
   - Key pair (login): select your .pem keypair
   - Network settings: Select existing security group, then select your group from the list
   - Advanced details: IAM instance profile > select your profile
4. Select **Create launch template**.

Next, create an Auto Scaling group - a logical group of instances that are related to the overall application:
- A target group is a logical grouping of EC2 instances that server a common goal.

1. Go to **EC2** > **Auto Scaling Groups**.
2. Select **Create Auto Scaling Group**.
3. Add a name, and select a launch template.
4. In **Network**, accept the **VPC** default, and select the first option in the **Availability Zones and subnets** list.
5. In **Load balancing**, select **Attach to a new load balancer**.
6. In **Attach to a new load balancer**, select the following:
   - Select **Appliction Load Balancer**
   - Enter a **Load balancer name**
   - Select two options in **Availability Zones and subnets**. Your default zone is already selected, so select one more.
   - In **Default routing (forward to)**, select **Create a target group**. 
     - Enter a name or accept the default.
   Select **Next**.
7. On **Configure group size and scaling**, you set the min and max number of instances you want to run, depending on scaling. Leave these at **1** for the test env.
8. Select **Skip to review**.
9. Select **Create Auto Scaling group**.

When the group is created, it will spin up a new instance because we set our minimum capacity at **1**.

Access the website at the DNS name for the load balancer (**Load Balancing** > **Load Balancers**)

### Costs

Your expenses are listed in the **Billing and Cost Management** dashboard:
- Manage with root account
- Search for **Billing** to open **Billing and Cost Management**.
- Add a billing alert
- Delete unneeded backups
- Only run EC2 instances when needed






### Commands

```bash
ssh -i /path/to/ec2-key.pem ubuntu@<public-instance-ip>         # ssh into machine
```