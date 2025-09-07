---
title: "xAnsible"
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
apt install ansible                                 # 1. install package - make sure ansible 2.16+
adduser ansible                                     # 2. create ansible user
usermod -aG sudo ansible                            # 3. make ansible sudoer
ssh-copy-id -i ~/.ssh/id_rsa.pub ansible@<ip-addr>  # 4. copy ssh keys to ansible user acct
vim /etc/sudoers.d/ansible                          # 5. add ansible file
ansible ALL=(ALL) NOPASSWD: ALL                     # 6. add to ansible sudo file
chown root:root /etc/sudoers.d/ansible              # 7. root perms to file
chmod 440 /etc/sudoers.d/ansible                    # 8. update perms
ssh <client-ip> sudo ls /etc                        # 9. test from ansible acct on master machine
```
### Inventory files

Look into _roles_ in ansible.

An inventory file tells Ansible where to find the servers to configure:
- create the dir `/etc/ansible`
- create the inventory file `/etc/ansible/hosts`
- give perms to ansible account
- config file is `/etc/ansible/ansible.cfg`
  - you can fine tune performance with this file
  - ansible reads from this config file every time it runs
  - list where to find host file and default user
  - ansible reads the current dir for this file each time
  - can store it in git repo, but discouraged bc might include sensitive info
- 


```bash
mkdir /etc/ansible                  # 1. make config dir
touch /etc/ansible/hosts            # 2. make inventory file
chown ansible /etc/ansible/hosts    # 3. make ansible user the owner
chmod 600 /etc/ansible/hosts        # 4. only owner has rw perms
vim /ect/ansible/hosts              # 5. add host IPs or DNS names to file
vim /etc/ansible/ansible.cfg        # 6. create config file
[defaults]                              # start of stanza
inventory = /etc/ansible/hosts          # path to inventory file
remote_user = normaluser                # default user for playbooks - must exist on client machines
ansible all -m ping                 # 7. test that ansible can connect to clients via SSH
```

### Playbooks

Ansible stores configurations in a _playbook_, a YAML-formatted file that contains ansible instructions:
- each individual instruction is a _play_
- `ansible.builtin.apt` is an ansible module for `apt` - it tells ansible we are using a native module, not an external tool
  - There is an `ansible.builtin.dnf` too
- Ansible can pull config files from online repos:
  - configure cron job to apply changes to configs

```bash
# --- Install htop example --- #
---
- hosts: all                    # the hosts you want the playbook to apply to (all in inventory file)
  become: true                  # use sudo to execute commands in this playbook
  tasks:                        # new section that contains tasks ('plays')
  - name: Install htop          # name of task
    ansible.builtin.apt:        # tell apt module to install htop
      name: htop

# --- Install multiple packages example --- #
---
- hosts: all
  become: true
  tasks:
  - name: Install multiple packages
    ansible.builtin.apt:
      name:
        - git
        - vm-nox
        - htop

# --- Copy files example --- #
---
- hosts: all
  become: true
  tasks:
  - name: copy SSH motd
    ansible.builtin.copy:   # copy message of the day (motd) file to inventory
      src: motd
      dest: /etc/motd
```

### Web server playbook

This playbook:
- Installs Apache
- Starts the service with `service` module
- Copies an HTML index file

```bash
# --- Sample file --- #
---
- hosts: all
  become: true
  tasks:
  - name: Install Apache
    ansible.builtin.apt:
      name: apache2
  - name: Start the apache2 service
    ansible.builtin.service:
      name: apache2
      state: started
      enabled: true

  - name: Copy index.html
    ansible.builtin.copy:
      src: index.html
      dest: /var/www/html/index.html

```

### Practice

- Install security updates
- create user accts
- set passwords
- enable services to automatically start at boot

### Pull method

Inventory files work well in small environments with static IPs, but envs where machines regularly become decommissioned and IPs change--the cloud--this might not work as well.

In _pull mode_, there is no central server--each server runs ansible against itself.
- Keep playbooks in a git repo that is accessible from managed servers
- Main playbook name is `local.yml`. Otherwise, you have to specifiy it in the command
- `hosts:` must be `localhost`
- Must run commands as ansible user or user with sudo bc commands are not interactive
- Runs tasks/plays that haven't been config'd yet, skips steps that exist on server
  - `o` option only runs if there have been changes to the playbook in the repo

Use cases:
- Can create a playbook per server type: web server, db server, file server, etc
- Create cloud server that runs ansible each time it is provisioned
  - Install ansible first
  - then `ansible-pull`



```bash
ansible-pull -U https://github.com/<username>/<repo-name>.git   # run pull method - U is URL option
ansible-pull -U https://github.com/<username>/<repo-name>.git   # run pull method - U is URL option

# --- Sample file --- #
---
- hosts: localhost                      # execute commands locally, not via SSH
  become: true
  tasks:
  - name: Install Apache
    ansible.builtin.apt:
      name: apache2
  - name: Start the apache2 service
    ansible.builtin.service:
      name: apache2
      state: started
      enabled: true

  - name: Copy index.html               # this file must be in root of repo
    ansible.builtin.copy:
      src: index.html
      dest: /var/www/html/index.html
```