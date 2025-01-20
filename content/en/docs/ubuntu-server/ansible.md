---
title: "Ansible"
# linkTitle: ""
weight: 130
# description:
---

Provides configuration management and automated deployments:
- IaC - Infrastructure as code - configuration management - automated running of scripts on your servers
  - Quickly deploy infrastructure
  - Make changes to all existing servers
  - Ex: single command to setup web server with req'd apps, config files, users, firewall rules
- Some tools use a program or agent reads instructions from a central server and runs them on clients
  - Require CPU and RAM on central server and client servers for the agent
- admin can write a script and execute it across all servers
- Tools include Ansible, Chef, Puppet, etc

## Ansible

Ansible uses a server but doesn't use an agent:
- Uses SSH
- Special ansible user acct on each system, and server (or admin workstation) runs commands via SSH to update config
- No agent, so uses much less CPU
- Scales better and provides more granular control than other IaC solutions

Inventory file that lists resources (servers) with ansible user account:
- can store inventory file in git repo - called the **pull method**
- main workstation with ansible installed
  - Clients need openssh

1. Create ansible user on central server and all clients
2. Setup passwordless ssh on each client
3. give ansible user sudo privs


### Setup ansible client

```bash
apt install ansible                                 # 1. install package
adduser ansible                                     # 2. create ansible user
usermod -aG sudo ansible                            # 3. make ansible sudoer
ssh-copy-id -i ~/.ssh/id_rsa.pub ansible@<ip-addr>  # 4. copy ssh keys to ansible user acct
vim /etc/sudoers.d/ansible                          # 5. add ansible file
ansible ALL=(ALL) NOPASSWD: ALL                     # 6. add to ansible sudo file
chown root:root /etc/sudoers.d/ansible              # 7. root perms to file
chmod 440 /etc/sudoers.d/ansible                    # 8. update perms
ssh <client-ip> sudo ls /etc                        # 9. test from ansible acct on master machine
```


Host u24
	HostName 192.168.56.50
	User linuxuser
	Port 22

Host ---
	HostName 192.168.56.51
	User normaluser
	Port 22

Host us24
	HostName 192.168.56.52
	User normaluser
	Port 22

Host db2
	HostName 192.168.56.53
	User maria
	Port 22

Host ans1
	HostName 192.168.56.54
	User normaluser
	Port 22

Host ans2
	HostName 192.168.56.55
	User normaluser
	Port 22
