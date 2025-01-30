---
title: "Terraform cloud deployments"
linkTitle: "Terraform"
weight: 180
# description:
---

Automated deployments have lots of benefits:
- Saves time
- reduces possibility of human error
- disaster recovery - you can recover resources much faster
- living blueprint - another admin can look at the automation script and get a good understanding of the infrastructure


Terraform can automate cloud infrastructure:
- Infrastructure as code - write code that represents the desired end state
  - Download it to your machine and it turns script files into infrastructure
- Cross platform - can run on any cloud provider
- Should store in a git repo
- Made by Hashicorp
- Lower-level than ansible or puppet

## Installation

You have to download Terraform, then configure AWS as the root user:

1. Go to https://www.terraform.io/ and copy the commands to install
2. Log into your AWS console as root
3. Go to **IAM** > **Users**
4. Select **Create User**
5. Enter a name, and do not allow access to the AWS Management Console
6. On **Set Permissions**, select **Attach policies directly**, and then select the box beside **AdministratorAccess** in the **Permissions policies** list. This policy grants Terraform access to AWS.
7. On the next page, skip the settings and select **Create User**.

## Create basic EC2 instance

- Go to https://cloud-images.ubuntu.com/locator/ec2/ to find Ubuntu AMI IDs

1. Create a directory to store your terraform config files.
2. Create a file with a `.tf` extension:
    ```bash
    provider "aws" {                                                # provider
        profile = "profile-name"                                    # if you have creds for more than one aws profile
    	region = "us-east-1"
    }

    resource "aws_instance" "my-server-1" {                         # resource block, "values" here are specific to AWS
    	ami				            = "ami-04b4f1a9cf54c11d0"       # AMI ID
    	associate_public_ip_address	= "true"                        # give the instance a public IP
    	instance_type			    = "t2.micro"                
    	key_name			        = "production-key"              # key pair - look in EC2 > Resources > Key pairs
    	vpc_security_groups_ids		= [ "sg-000511aaefb840967" ]    # security group - EC2 > Network & Security > Security Groups
    	tags = {                                                    # custom tags
    		Name = "Web Server 1"                                   # name of EC2 instance in list
    	}
    }
    ```
3. Export environment variables using the keys for the terraform user:
   ```bash
   export AWS_ACCESS_KEY_ID="<key>"
   export AWS_SECRET_ACCESS_KEY="<key>"
   ```
4. Change to a directory with terraform.tf files, and run the init command:
   ```bash
   cd config/
   terraform init
   ```
5. Check the syntax on your config file:
   ```bash
   terraform plan
   ```
6. If the plan command succeeds with no errors, you can apply the config file:
   ```bash
   terraform apply
   ```