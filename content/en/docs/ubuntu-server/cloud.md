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

