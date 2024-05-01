---
title: "Orchestration"
weight: 240
---

Orchestration is the organization of a process that is balanced and coordinated and that achieves consistent results. Some processes that you can orchestrate:
- App development
- Configuration management
- Disaster recovery
- Server monitoring
- Security

## Devops

Developer operations, improves software delivery. It is a combo of a developer and system admin. Includes the following:
- Continuous integration
- Continuous testing
- Continuos delivery or deployment
- Infrastructure as code
- Infrastructure automation
- Monitoring and logging

DevOps is about continually providing new software features and fixes to the customer through small changes to the app:

Continual app processing
: Sets up a pipeline to always deliver software to the customer:
  - **Continuous integration**: Use software revision control to quickly integrate app changes into the main software branch.
  - **Continuous testing**: Automate testing to avoid breaking the app when a branch merges
  - **Continuous delivery**: Deliver software to the customer on a continual basis.

Controlling app environment
: Dev and prod environments (infrastructure) must match. The environment must be controlled and tested like the app environment, so you can rollback changes that cause issues.

Defining the app environment
: Infrastructure as code implements non-hardware specifications with automated code, called configuration management. This includes security measures, like firewalls and authentication policies. This is called _policy as code_.

  IaC provides repeatable, versionable environments.

Deploying the app environment
: Move an app and its environment to prod on a regular basis with infrastructure automation. Use tools like Red Hat's Ansible.

Monitoring app environments
: In prod, the app should monito and log software metrics, resource usage, performance stats, etc.

### Containers

Containers are a big part of DevOps, and provide the following benefits:
- **Static environment**: The container image is predetermined and immutable.
- **Version control**: After development and before prod, you can use VCS to version your container and store it alongside previous container versions.
- **Replace not update**: In the production environment, you do not update the deployed app--you stop the running container, and then replace it with the new container version.
- **High availability**: You can easily replicate the production app, which means you can stop the old prod app and replace it with the new prod app version for continual uptime. No need to shut down your prod app at specified times.

## Provisioning


### Coding infrastructure

Container infrastructure is managed and controlled much like software revisions:
- **Determine the infrastructure**: Determine the OS, libraries, services, security config, etc and make it immutable.
- **Document the infrastructure**: Preset app container infra is doc'd through an orchestration tool, and then loaded into the utilities code portal. This is called _automated configuration management_. Use this for _build automation_, which is when you deploy and replicate container image infrastructure.
- **Provide revision control**: The orchestration tool tracks each change when you add the app to the tool's registry.
- **Troubleshooting**: Check the configuration and use the orchestration tool's registry utilities to find bugs and errors.

### Automate infrastructure

Orchestration tools and automated configuration management let you replicate the prod app container without manual intervention. Here are some tools:
- **Ansible**: Red Hat product that uses OpenSSH and Python to remotely configure infrastructure withouth needing a running agent on remove hosts. Uses JSON-based protocols and a plaintext config file.
- **Chef**: Ruby-based, uses "recipes" to define server configurations.
- **Puppet**: Uses its own language to define system configs for remote servers.
- **SaltStack**: Owned by VMware, Python-based configuration management tool that uses YAML-formatted config files.
- **Terraform**: HashiCorp, uses its own declarative language for storing server configs in JSON format. Lets you graph resources.

### Agent and agentless

Monitoring, logging, and reporting might adversely affect a container's health.
- _Agent monitoring tools_ require softare (agent) to be installed in the app being monitored to collect data and transmit it to another location like a monitoring server. Some say this affects the app performance.
- _Agentless monitoring tools_ does not require software (agent), it uses preexisting embedded software in the container to container environment to conduct its monitoring activity.

### Investigating inventory

Many orchestration platforms are _self-healing_. You have a desired state, and if there is an issue that causes changes to that state, the orchestration tool works to correct issues and restore the desired state.

## Container orchestration engines

### Kubernetes

Open source, de facto standard that is highly scalable and fault tolerant. Each service or app has the following components:
- Cluster service: YAML files to deploy and manage apps
- Pod: Runs one or more containers
- Worker: Pod host system that uses the kublet agent to communicate with cluster services
- YAML file: Defines the app's desired state and config

### Docker Swarm

A group of docker containers is referred to as a cluster, which appears to the user as a single container. Docker Swarm orchestrates this docker cluster.
- Can monitor cluster health and return to desired state
- Deploy additional docker containers as needed
- Faster than K8s

Swarm is often used by people that know docker to learn orchestration concepts.

### Mesos

Apache Mesos is not an orchestration platform, it is a distributed systems kernel:
- Can create containers
- When combined with Marathon, it can do some container orchestration
- Provides high availability and health monitoring integrations