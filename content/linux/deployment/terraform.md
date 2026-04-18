+++
title = 'Terraform'
date = '2025-09-07T19:08:20-04:00'
weight = 10
draft = false
+++



Terraform automates cloud infrastructure by letting you write code that describes your desired end state. When you run Terraform, it reads your configuration files and provisions the corresponding infrastructure. Store configuration files in a git repository so other administrators can read the scripts and understand the infrastructure.

Terraform is lower-level than Ansible or Puppet and works across cloud providers. It is made by HashiCorp.

Two block types control network access on a resource:

- **Ingress** blocks allow incoming connections on a specified port from specified IP addresses.
- **Egress** blocks allow the instance to connect to external networks.

Automated infrastructure deployments offer several advantages over manual processes:

- Reduce the risk of human error during provisioning.
- Speed up disaster recovery by recreating resources from code.
- Provide a living blueprint — another administrator can read the configuration and understand the infrastructure.

## Installation

Install Terraform and create a dedicated AWS IAM user that Terraform uses to provision resources:

1. Go to [terraform.io](https://www.terraform.io/) and follow the instructions to install Terraform on your machine.
2. Log into your AWS console as root.
3. Go to **IAM** > **Users**.
4. Select **Create User**.
5. Enter a name. Do not allow access to the AWS Management Console.
6. On **Set Permissions**, select **Attach policies directly**, then select **AdministratorAccess** from the **Permissions policies** list. This grants Terraform the access it needs to provision resources.
7. Skip the remaining settings and select **Create User**.

## Create a basic EC2 instance

Find Ubuntu AMI IDs at [cloud-images.ubuntu.com/locator/ec2](https://cloud-images.ubuntu.com/locator/ec2/). To create a basic EC2 instance:

1. Create a directory to store your Terraform configuration files.
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
3. Export the IAM user's access keys as environment variables:
   ```bash
   export AWS_ACCESS_KEY_ID="<key>"
   export AWS_SECRET_ACCESS_KEY="<key>"
   ```
4. Change to the directory containing your `.tf` files and initialize Terraform:
   ```bash
   cd config/
   terraform init
   ```
5. Validate the configuration:
   ```bash
   terraform plan
   ```
6. Apply the configuration:
   ```bash
   terraform apply
   ```


### Adding security

Define a security group as a separate resource and reference it by variable in the instance block. This lets you manage firewall rules independently from the instance and reuse the group across multiple resources:

```bash
provider "aws" {                            
	profile = "terraform-provisioner"
	region = "us-east-1"
}

resource "aws_instance" "my-server-1" {
	ami				            = "ami-0684e5b823fb79036"
	associate_public_ip_address	= "true"
	instance_type			    = "t2.micro"
	key_name			        = "production-key"
	vpc_security_group_ids		= [aws_security_group.external_access.id]   # variable for security group created below
	tags = {
		Name = "Web Server 1"
	}
}
	resource "aws_security_group" "external_access" {   # create new security group resource named in tf as "external_access"
		name		= "my_sg"                           # sg group name in AWS
		description	= "Allow OpenSSH and Apache"        
		ingress {                                       # allow SSH connections on port 22 from this IP
			from_port	= 22
			to_port		= 22
			protocol	= "tcp"
			cidr_blocks	= [ "172.11.59.105/32" ]
			description	= "Home Office IP"
		}
		ingress {                                       # allow HTTP connections on port 80 from this IP
			from_port	= 80
			to_port		= 80
			protocol	= "tcp"
			cidr_blocks	= [ "172.11.59.105/32" ]
			description	= "Home Office IP"
		}
		egress {                                        # allow public internet access (any IP)
			from_port	= 0
			to_port		= 0
			protocol	= "-1"
			cidr_blocks	= ["0.0.0.0/0"]
		}
	
	}
```

### Destroying resources

Run `terraform destroy` to remove all resources defined in a configuration file. Use this command when resources are no longer in use or when you need to reset an environment:

```bash
terraform destroy
```


### Terraform and Ansible

Use Terraform to provision the initial server and infrastructure, then use Ansible to automate ongoing configuration. Terraform can launch the initial Ansible run so you only need to execute a single script.

To combine these tools, create a shell script in the same directory as your `.tf` file and reference it in the Terraform configuration as `user_data`. AWS runs the `user_data` script automatically when the instance is created.

#### bootstrap.sh

This script installs Ansible on the new instance and runs `ansible-pull` to retrieve and apply `local.yaml` from the repository:

```bash
#!/bin/bash
sudo apt update
sudo apt install -y ansible
sudo ansible-pull -U https://github.com/rjseymour66/ansible.git
```

#### Terraform config

Reference `bootstrap.sh` in the `user_data` field of the instance resource:

```bash
provider "aws" {
	profile = "terraform-provisioner"
	region = "us-east-1"
}

resource "aws_instance" "my-server-1" {
	ami				= "ami-0684e5b823fb79036"
	associate_public_ip_address	= "true"
	instance_type			= "t2.micro"
	key_name			= "production-key"
	vpc_security_group_ids		= [aws_security_group.external_access.id]
	user_data = file("bootstrap.sh")                                            # script ref
	tags = {
		Name = "Web Server 1"
	}
}
	resource "aws_security_group" "external_access" {
		name		= "my_sg"
		description	= "Allow OpenSSH and Apache"
		ingress {
			from_port	= 22
			to_port		= 22
			protocol	= "tcp"
			cidr_blocks	= [ "108.26.179.236/32" ]
			description	= "Home Office IP"
		}
		ingress {
			from_port	= 80
			to_port		= 80
			protocol	= "tcp"
			cidr_blocks	= [ "108.26.179.236/32" ]
			description	= "Home Office IP"
		}
		egress {
			from_port	= 0
			to_port		= 0
			protocol	= "-1"
			cidr_blocks	= ["0.0.0.0/0"]
		}
	
	}

```