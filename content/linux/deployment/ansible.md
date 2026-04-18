+++
title = 'Ansible'
date = '2025-09-07T19:07:51-04:00'
weight = 10
draft = false
+++


Ansible is an infrastructure-as-code (IaC) tool for configuration management and automated deployments. Use it to install packages, manage config files, create users, and apply firewall rules across all your servers with a single command.

Unlike tools such as Chef and Puppet, Ansible does not require an agent on each client. It connects over SSH, which reduces CPU and RAM overhead and scales better across large environments. A dedicated `ansible` user account on each managed server handles remote execution.

Ansible reads server addresses from an *inventory file*. You can store the inventory file in a git repository and use *pull mode* to have each server configure itself. Client machines require OpenSSH.

## Setup

Complete this one-time setup on the central server and each client before using Ansible:

1. Create the `ansible` user on the central server and all clients:
   ```bash
   adduser ansible
   ```
2. Set up passwordless SSH from the central server to each client:
   ```bash
   ssh-copy-id -i ~/.ssh/id_rsa.pub ansible@<client-ip>
   ```
3. Grant the `ansible` user passwordless sudo privileges:
   ```bash
   vim /etc/sudoers.d/ansible
   ```
   Add:
   ```bash
   ansible ALL=(ALL) NOPASSWD: ALL
   ```
   Then set the correct ownership and permissions:
   ```bash
   chown root:root /etc/sudoers.d/ansible
   chmod 440 /etc/sudoers.d/ansible
   ```
4. Test that the central server can connect and run sudo commands on a client:
   ```bash
   ssh <client-ip> sudo ls /etc
   ```
## Inventory files

The inventory file tells Ansible which servers to manage. The configuration file (`ansible.cfg`) tells Ansible where to find the inventory and which user to connect as. Ansible reads `ansible.cfg` from the current directory each time it runs. You can store it in a git repository, but avoid including sensitive information such as passwords or private keys.

To create the inventory and configuration files:

1. Create the configuration directory:
   ```bash
   mkdir /etc/ansible
   ```
2. Create the inventory file and set ownership:
   ```bash
   touch /etc/ansible/hosts
   chown ansible /etc/ansible/hosts
   chmod 600 /etc/ansible/hosts
   ```
3. Add server IP addresses or DNS names to the inventory file:
   ```bash
   vim /etc/ansible/hosts
   ```
4. Create the configuration file:
   ```bash
   vim /etc/ansible/ansible.cfg
   ```
   Add:
   ```bash
   [defaults]
   inventory = /etc/ansible/hosts      # path to inventory file
   remote_user = ansible               # default user for playbooks — must exist on client machines
   ```
5. Test that Ansible can connect to all clients:
   ```bash
   ansible all -m ping
   ```

## Playbooks

A *playbook* is a YAML file containing Ansible instructions. Each individual instruction is a *play*. Playbooks use built-in modules to perform tasks — `ansible.builtin.apt` wraps the `apt` package manager, and `ansible.builtin.dnf` wraps `dnf`. You can also configure a cron job to pull and apply playbook changes from a remote repository on a schedule.

Run a playbook with:

```bash
ansible-playbook <playbook.yml>
```

#### Install a single package

```yaml
---
- hosts: all
  become: true
  tasks:
  - name: Install htop
    ansible.builtin.apt:
      name: htop
```

#### Install multiple packages

```yaml
---
- hosts: all
  become: true
  tasks:
  - name: Install multiple packages
    ansible.builtin.apt:
      name:
        - git
        - vim-nox
        - htop
```

#### Copy a file

```yaml
---
- hosts: all
  become: true
  tasks:
  - name: Copy SSH motd
    ansible.builtin.copy:
      src: motd
      dest: /etc/motd
```

## Web server playbook

This playbook installs Apache, starts and enables the service, and deploys an HTML index file. Use it as a starting point for provisioning web servers across your inventory:

```yaml
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

## Common tasks

These playbook examples cover frequently repeated administration tasks.

#### Install security updates

```yaml
---
- hosts: all
  become: true
  tasks:
  - name: Install security updates
    ansible.builtin.apt:
      upgrade: dist
      update_cache: true
```

#### Create a user account

```yaml
---
- hosts: all
  become: true
  tasks:
  - name: Create user account
    ansible.builtin.user:
      name: <username>
      state: present
```

#### Set a password

```yaml
---
- hosts: all
  become: true
  tasks:
  - name: Set user password
    ansible.builtin.user:
      name: <username>
      password: "{{ '<password>' | password_hash('sha512') }}"
```

#### Enable a service at boot

```yaml
---
- hosts: all
  become: true
  tasks:
  - name: Enable service at boot
    ansible.builtin.service:
      name: <service>
      enabled: true
```

## Pull method

The inventory file approach works well in small environments with static IP addresses. In dynamic environments such as the cloud, where machines are regularly provisioned and decommissioned, pull mode is more practical.

In pull mode, there is no central server. Each server runs Ansible against itself, pulling its playbook from a git repository. This makes pull mode well-suited for cloud instances that configure themselves on first boot.

Key requirements for pull mode:

- Store playbooks in a git repository accessible from all managed servers.
- Name the main playbook `local.yml`. If you use a different name, specify it in the command.
- Set `hosts:` to `localhost` — commands run locally, not over SSH.
- Run as the `ansible` user or a user with sudo privileges, since commands are non-interactive.

The `-o` flag skips execution if the repository has not changed since the last run:

```bash
ansible-pull -U https://github.com/<username>/<repo-name>.git      # pull and apply the playbook
ansible-pull -o -U https://github.com/<username>/<repo-name>.git   # only run if the repo has changed
```

#### Sample pull mode playbook

```yaml
---
- hosts: localhost
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